---
title: Blinking
type: docs
prev: assignments/SpinningAndBlinking/Arduino
weight: 1
---

Making an LED blink is rather straight forward:

```c
const unsigned int LED{17}; // define a constant for the LED pin

void setup() {
    pinMode(LED, OUTPUT); // configure the LED pin to be an output
}

void loop() {
    digitalWrite(LED, HIGH); // turn the LED on
    delay(1000); // wait 1 second
    digitalWrite(LED, LOW); // turn the LED off
    delay(1000); // wait 1 second
}
```

The `setup` function runs once on boot.

The `loop` function runs over and over again forever.
