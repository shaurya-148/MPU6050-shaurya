---
title: Firmware
type: docs
prev: assignments/full-stack/ble
next: assignments/full-stack/ble/app
weight: 1
---

## Setup

Create a new project with:

```sh
cargo embassy init {project_name}
```

Then run:

```sh
cargo add esp-wifi --features esp32s3,ble,async,phy-enable-usb
```

to add the `esp-wifi` crate with BLE capability enabled.

And add the following dependency in `Cargo.toml` under `[dependencies]`:

```toml
bleps = { git = "https://github.com/bjoernQ/bleps", package = "bleps", branch = "main", features = [
    "macros",
    "async",
] }
```
> This crate provides a simple hardware agnostic BLE interface.

Then append the following to `Cargo.toml`:

```toml
[profile.dev.package.esp-wifi]
opt-level = 3
```
> The `esp-wifi` crate must be built with performance optimizations prioritized as radio interaction is time-critical.

Then add an additional linker arg in `build.rs`:

```diff
fn main() {
    println!("cargo:rustc-link-arg-bins=-Tlinkall.x");
+   println!("cargo::rustc-link-arg=-Trom_functions.x")
}
```

---

To make sure everything is ok, run `cargo check --release`.

## GATT Server

With that taken care of, we can start introducing BLE into our firmware.

The BLE interface will require a timer for high level async operation, so
let's add another timer group to the system initialization:

```diff
esp_println::logger::init_logger_from_env();

let peripherals = Peripherals::take();
let system = SystemControl::new(peripherals.SYSTEM);
let clocks = ClockControl::max(system.clock_control).freeze();

let timg0 = TimerGroup::new(peripherals.TIMG0, &clocks);
+ let timg1 = TimerGroup::new(peripherals.TIMG1, &clocks);

esp_hal_embassy::init(&clocks, timg0.timer0);
```

Keep the LED initialization as is.

Then initialize the radio:

```rust
let init = esp_wifi::initialize(
    EspWifiInitFor::Ble,
    timg1.timer0,
    Rng::new(peripherals.RNG),
    peripherals.RADIO_CLK,
    &clocks,
)
.unwrap();
```

And constrain the BLE peripheral:

```rust
let mut bluetooth = peripherals.BT;

let connector = BleConnector::new(&init, &mut bluetooth);
let mut ble = Ble::new(connector, esp_wifi::current_millis);
println!("Connector created");
```

Then create a loop for connecting and serving (you can replace the contents
of the already present blink loop):

```rust
loop {
    // we're going to be putting stuff here
}
```

We can initialize BLE:

```rust
println!("{:?}", ble.init().await);
```
> The `println!` is purely for debugging purposes.

The first step of initiating a BLE connection is advertisement.

It is the obligation of BLE peripherals to advertise themselves.
Advertisement data may contain manufacturer information, available services,
device name, etc.

{{< callout type="info" >}}
  Refer to `Core Specification Supplement -  Section 1` for more information on
  advertisement data types.
{{< /callout >}}

We can create an advertisement like so:

```rust
println!("{:?}", ble.cmd_set_le_advertising_parameters().await);
println!(
    "{:?}",
    ble.cmd_set_le_advertising_data(
        create_advertising_data(&[
            AdStructure::Flags(LE_GENERAL_DISCOVERABLE | BR_EDR_NOT_SUPPORTED),
            AdStructure::ServiceUuids128(&[Uuid::Uuid128([
                0x38, 0xcf, 0x62, 0x0a, 0xc3, 0xfb, 0x10, 0x9f, 0xeb, 0x11, 0x54, 0x23,
                0xe0, 0x12, 0x73, 0x93
            ])]),
            AdStructure::CompleteLocalName("ece196"),
        ])
        .unwrap()
    )
    .await
);
println!("{:?}", ble.cmd_set_le_advertise_enable(true).await);

println!("started advertising");
```

This advertisement data indicates that a single service with a 128bit UUID (Universal Unique Identifier)
is available and that the device's name is "ece196".

Services and characteristics have UUIDs to uniquely identify themselves.
It makes sense to advertise the UUID of an available service.

In this case, I randomly created a UUID with [this](https://www.uuidgenerator.net) tool.

Now let's create a GATT server with a single service that has one characteristic:

```rust
gatt!([service {
    uuid: "937312e0-2354-11eb-9f10-fbc30a62cf38",
    characteristics: [characteristic {
        uuid: "937312e0-2354-11eb-9f10-fbc30a62cf38",
    },],
},]);

println!("{:?}", gatt_attributes);
```
> Notice the byte-order is reversed. This is part of the BLE spec.

This is pretty useless though, since the characteristic doesn't hold any value.

Characteristics can support any and all of the following accesses:

1. Read - Client can read the characteristic value.
1. Write - Client can write the characteristic value.
1. Indicate - Client can spontaneously become aware of a new characteristic value.
1. Notify - Client can spontaneously become aware of a new characteristic value and **must** acknowledge.

Let's add read and write to our characteristic, to enable the control and monitoring of the LED state.

Create two callbacks, one for read and one for write, we can make closures:

```rust
let mut rf = |_offset: usize, data: &mut [u8]| {
    let buf = (led.is_set_high() as u8).to_be_bytes();
    data[..buf.len()].copy_from_slice(&buf);

    buf.len()
};

let mut wf = |_offset: usize, data: &[u8]| {
    if data.len() != 1 {
        panic!(
            "Invalid data received. Expected length 1, got {}",
            data.len()
        );
    }

    led.set_level(match data[0] {
        0 => Level::Low,
        1 => Level::High,
        val => {
            panic!("Invalid data received. Expected [0, 1], got {}", val);
        }
    });
};
```

The first callback `rf` is responsible for fetching the data requested and determining the data length.

In this case, we measure the LED state and copy it to the read buffer, then return the number of bytes used (`1`).

The second callback `wf` is responsible for taking action based on received data.

In this case, if the received data is a `0`, the LED is turned off. A `1`, and it's turned on.

This callback has two failure modes, length and value. If the received data is more than one byte *or* an unexpected value,
the client is not complying with the process we have established.

> Rather than panicing, what would be a more graceful approach?

Then you can update the `gatt` configuration with these callbacks:

```diff
gatt!([service {
    uuid: "937312e0-2354-11eb-9f10-fbc30a62cf38",
    characteristics: [characteristic {
        uuid: "937312e0-2354-11eb-9f10-fbc30a62cf38",
+       read: rf
+       write: wf
    },],
},]);
```

If you were to try and compile the firmware at this time, you would notice that it fails.

This is because these closures refer to `led`. The `rf` closure borrows `led` immutably, and `wf` borrows `led` mutably.

Since both closures exist at the same time, this would require an immutable and mutable borrow to exist at the same time as well.

This is forbidden in Rust, for good reason. You cannot have an immutable binding to a resource which may be mutated, the invariance
of the immutable binding would be broken.

So what can we do about this?

Well, we know that these two closures cannot *run* at the same time, since the GATT server runs on a single unpreemptable thread.

We can take advantage of **interior mutability** in this case.

Go back up to where `led` is created and do the following:

```diff
let led = Output::new(io.pins.gpio17, Level::Low);
+ let led_ref = RefCell::new(led);
+ let led_ref = &led_ref;
```

A `RefCell` is an interior mutability primitive that fascilitates *runtime* borrow checking.

So each time we try to access the LED we will *ask* at runtime if it is currently borrowed.

Back to the callbacks:

```rust
let mut rf = |_offset: usize, data: &mut [u8]| {
    let buf = (led_ref.borrow().is_set_high() as u8).to_be_bytes();
    data[..buf.len()].copy_from_slice(&buf);

    buf.len()
};

let mut wf = |_offset: usize, data: &[u8]| {
    if data.len() != 1 {
        panic!(
            "Invalid data received. Expected length 1, got {}",
            data.len()
        );
    }

    led_ref.borrow_mut().set_level(match data[0] {
        0 => Level::Low,
        1 => Level::High,
        val => {
            panic!("Invalid data received. Expected [0, 1], got {}", val);
        }
    });
};
```
> `borrow()` and `borrow_mut()` will panic if the resource is already borrowed, but we know
> the lifetimes of these closures will not overlap.
> The compiler may be able to determine this fact, and could optimize out the runtime check
> entirely!

Finally, we can start the server:

```rust
let mut rng = bleps::no_rng::NoRng;
let mut srv = AttributeServer::new(&mut ble, &mut gatt_attributes, &mut rng);

let mut notifier = || future::pending();

srv.run(&mut notifier).await.unwrap();
```

The `notifier` can be used to send notifications to the client when events occur,
but in this case we don't want to do that so we create an unresolvable future.

---

Now you can `cargo run --release` and open the NRF Connect app (get it from the app store).

Filter the scanner by typing in the name "ece196" and connect.

You will see the service and characteristic appear with a read and write button.

Try writing the number `1` and see the LED turn on!
