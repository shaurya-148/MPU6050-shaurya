---
title: Firmware
type: docs
prev: assignments/full-stack/ble
next: assignments/full-stack/ble/client
weight: 1
---

## Setup

Create a new project with:

```sh
cargo embassy init {project_name} --chip esp32c3
```

Then run:

```sh
cargo add esp-wifi --features esp32c3,ble
```

to add the `esp-wifi` crate with BLE capability enabled.

And add the following dependency:

```toml
bleps = { git = "https://github.com/bjoernQ/bleps", package = "bleps", rev = "a5148d8ae679e021b78f53fd33afb8bb35d0b62e", features = [
    "macros",
    "async",
] }
```
> This crate provides a simple hardware agnostic BLE interface.

Then add an additional linker arg in `build.rs`:

```diff
fn main() {
    println!("cargo:rustc-link-arg-bins=-Tlinkall.x");
+   println!("cargo::rustc-link-arg=-Trom_functions.x")
}
```

And like previously, patch the esp32 deps in `Cargo.toml`:

```toml
[patch.crates-io]
esp-hal = { git = "https://github.com/esp-rs/esp-hal/", rev = "f95ab0def50130a9d7da0ba0101c921e239ecdb5" }
esp-hal-embassy = { git = "https://github.com/esp-rs/esp-hal/", rev = "f95ab0def50130a9d7da0ba0101c921e239ecdb5" }
esp-backtrace = { git = "https://github.com/esp-rs/esp-hal/", rev = "f95ab0def50130a9d7da0ba0101c921e239ecdb5" }
esp-println = { git = "https://github.com/esp-rs/esp-hal/", rev = "f95ab0def50130a9d7da0ba0101c921e239ecdb5" }
esp-wifi = { git = "https://github.com/esp-rs/esp-hal/", rev = "f95ab0def50130a9d7da0ba0101c921e239ecdb5" }
```
