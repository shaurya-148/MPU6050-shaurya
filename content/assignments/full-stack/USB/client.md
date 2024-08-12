---
title: Client
type: docs
prev: assignments/full-stack/usb/firmware
weight: 2
---

Time for the other half...

This is going to be a lot, not gonna lie this is going to go pretty deep.

We need to write code that does the following:

- Renders the UI
- Handles the serial port
- Connects the UI and serial together

This may seem trivial at first, but you must realize that Python - by default - is single threaded. This means if we make a _Button_ that sends a message over serial, _by default_ the UI will be **frozen** until that serial code is done executing.

This is bad, we don't like this, so we will need to take care to design our app with _concurrency_ in mind.

## UI

Let's start easy and just set up a simple UI.

We will be using [Tkinter](https://docs.python.org/3/library/tkinter.html) for this app.

We start by creating the object that represents our Tkinter session:

```python
import tkinter as tk
import tkinter.ttk as ttk

class App(tk.Tk):
    def __init__(self):
        super().__init__()

        self.title("LED Blinker")

if __name__ == '__main__':
    app = App()
    app.mainloop()
```

Ok this is pretty neat, when you run this program you should see an empty window appear.

Let's add our UI elements (called "widgets" in Tkinter):

```python
def __init__(self):
    super().__init__()

    self.title("LED Blinker")

    ttk.Checkbutton(self, text='Toggle LED').pack()
    ttk.Button(self, text='Send Invalid').pack()
    ttk.Button(self, text='Disconnect', default='active').pack()
```

What a nice simple app.

We have:

- a checkbox to toggle the LED
- a button to test sending an invalid byte
- a disconnect button

## Backend

We now need to implement the missing backend components.

_How_ do we run code when the checkbox is checked or unchecked?

Well, `ttk.CheckButton` has two more kwargs: `variable` and `command`.

It will set the passed `variable` to the state of the checkbox on change, and call the passed `command` on change.

We need to create both of those though, let's start with the `variable`:

```python
self.led = tk.BooleanVar()

ttk.Checkbutton(self, text='Toggle LED', variable=self.led, command=self.update_led).pack()
ttk.Button(self, text='Send Invalid').pack()
ttk.Button(self, text='Disconnect', default='active').pack()
```

We create a member variable `led` that represents the state of the checkbox. It is a special Tkinter variable type that can be mutated and observed by Tkinter widgets.

{{< callout type="info" >}}
  We need `led` to be a member variable, so we can access it throughout the lifetime of our `App` instance, even after the constructor finishes.
{{< /callout >}}

We also need to create the function `self.update_led()`:

```python
def update_led(self):
    value = self.led.get()

    # send `value` somehow??
```

Hmm... so how can we send our message?

We need to have access to the serial port in our app. Well, just like `led`, we can make a member variable of type `Serial`.

We can change our `App` class:

```python
class App(tk.Tk):
    ser: Serial

    def __init__(self):
        ...
```

Oh... but, how do we connect to the serial device? We can't just hardcode `/your/port` because it changes all the time.

We need to create a menu the user can select their port from and then connect.

Let's make another class called `SerialPortal` that provides this functionality:

```python
from serial import Serial
from serial.tools.list_ports import comports

class SerialPortal(tk.Toplevel):
    def __init__(self, parent: App):
        super().__init__(parent)

        self.parent = parent
        self.parent.withdraw() # hide App until connected

        ttk.OptionMenu(self, parent.port, '', *[d.device for d in comports()]).pack()
        ttk.Button(self, text='Connect', command=self.connect, default='active').pack()

    def connect(self):
        self.parent.connect()
        self.destroy()
        self.parent.deiconify() # reveal App
```

...and add the new popup window, a `connect` function and `port` variable to `App`:

```python
class App(tk.Tk):
    ser: Serial

    def __init__(self):
        super().__init__()

        self.title("LED Blinker")

        self.port = tk.StringVar() # add this
        self.led = tk.BooleanVar()

        ttk.Checkbutton(self, text='Toggle LED', variable=self.led, command=self.update_led).pack()
        ttk.Button(self, text='Send Invalid').pack()
        ttk.Button(self, text='Disconnect', default='active').pack()

        SerialPortal(self) # and this

    # and finally this
    def connect(self):
        self.ser = Serial(self.port.get())

    def update_led(self):
        value = self.led.get()

        # send `value` somehow??
```

Ok cool, now we can fill in `update_led`:

```python
def update_led(self):
    self.ser.write(bytes([self.led.get()]))
```

At this point, checking the LED checkbox should toggle the LED!

But there is still more to do...

---

Let's finish the backend for the _Send Invalid_ and _Disconnect_ buttons:

```python
def disconnect(self):
    self.ser.close()

    SerialPortal(self) # display portal to reconnect

def send_invalid(self):
    self.ser.write(bytes([0x10]))
```

This is pretty great, but we haven't used the response from the DevBoard yet, you'll notice clicking the _Send Invalid_ button does nothing.

We should standardize writing to the serial port by making one `write` function that all callbacks use.

First add this import for displaying an alert:

```python
from tkinter.messagebox import showerror
```

Then define the same constants as in our firmware on the DevBoard:

```python
S_OK: int = 0xaa
S_ERR: int = 0xff
```

And finally add the `write` function to `App`:

```python
def write(self, b: bytes):
    try:
        self.ser.write(b)
        if int.from_bytes(self.ser.read(), 'big') == S_ERR:
            showerror('Device Error', 'The device reported an invalid command.')
    except SerialException:
        showerror('Serial Error', 'Write failed.')
```

Now we can change our callbacks that wrote to the serial port to use this:

```python
def update_led(self):
    self.write(bytes([self.led.get()]))

def send_invalid(self):
    self.write(bytes([0x10]))
```

And now these will properly handle an `S_ERR` response.

## Safe Resource Acquisition

Another thing we need to do is make sure our serial port is properly closed when our app exits.

We can achieve this by allowing `App` to be constructed in a managed context.

This is what the [with](https://docs.python.org/3/reference/compound_stmts.html#with) keyword is for in Python.

To add this support to `App`, we add these functions:

```python
def __enter__(self):
    return self

def __exit__(self, *_):
    self.disconnect()
```

And change our main code to guarantee resource release:

```python
if __name__ == '__main__':
    with App() as app:
        app.mainloop()
```

## Threading

Right now, if any serial code gets stuck or just takes a long time, our UI will freeze for that time.

To avoid this, we need to make sure all registered callbacks spawn as a [detached thread](https://www.youtube.com/watch?v=-i8Kzuwr4T4).

We can make a helper [decorator](https://peps.python.org/pep-0318/) that we can use to mark any functions we want to be spawned in a detached thread:

```python
from threading import Thread, Lock # we'll use Lock later ;)

def detached_callback(f):
    return lambda *args, **kwargs: Thread(target=f, args=args, kwargs=kwargs).start()
```

> Don't worry too hard about understanding this decorator, just know it coerces the function it's applied to into being spawned _into a thread_ upon invocation.

This decorator can be applied to our callbacks like so:

```python
@detached_callback
def update_led(self):
    self.write(bytes([self.led.get()]))
```

We have one final problem to solve...

It is now possible that multiple threads could try to use the serial port at once, this would cause [undefined behavior](https://en.wikipedia.org/wiki/Undefined_behavior) in the form of a [race condition](https://en.wikipedia.org/wiki/Race_condition).

To solve this, we need to wrap the `Serial` object in a [lock](https://docs.python.org/3/library/threading.html#threading.Lock).

We can do this by creating a new type, `LockedSerial`:

```python
class LockedSerial(Serial):
    _lock: Lock = Lock()

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

    def read(self, size=1) -> bytes:
        with self._lock:
            return super().read(size)

    def write(self, b: bytes, /) -> int | None:
        with self._lock:
            super().write(b)

    def close(self):
        with self._lock:
            super().close()
```

Our custom type inherits from `Serial` and overrides the member functions we use with a lock acquisition of the super's implementation.

Effectively, our type behaves _exactly like the `Serial` type_ but with a lock around every function call.

Now replace every use of the `Serial` type with `LockedSerial`.
