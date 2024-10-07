---
title: Setup
type: docs
prev: assignments/spinning-and-blinking/rust
weight: 1
---

{{< callout type="info" >}}
  Ideally these steps would be omitted and a Docker image would be provided.
  Unfortunately the environment we are setting up relies on USB access
  which is currently unsupported on Apple Silicon. Learn more [here](https://github.com/docker/for-mac/issues/5263).
{{< /callout >}}


## Install Rust

To install rust, follow the instructions [here](https://www.rust-lang.org/tools/install).

## Editor

There are many editors to choose from, here's some:

- [VSCode](https://code.visualstudio.com) (with the rust-analyzer extension)
- [Zed](https://zed.dev) ✨
- [RustRover](https://www.jetbrains.com/rust/) (if you like jetbrains)
- Vim (with tree-sitter and rust-analyzer)

## Embedded Tools

For ESP32s, we need to install ESP's tools.

Execute:

```sh
cargo install espup espflash
espup install
```

{{% details title="macOS/Linux" closed="true" %}}
  Execute...

  ```
  . $HOME/export-esp.sh
  ```

  ...in every terminal session, or append it to your shell profile.
{{% /details %}}

Now we need to install `cargo-embassy`, a tool for generating Embassy projects.

Execute:

```sh
cargo install cargo-embassy --git https://github.com/ece-196/cargo-embassy.git
```

## Project

It's time to create our Rust project in the assignment repository. Make sure you are in the
root directory of your assignment repo, and invoke cargo-embassy like so:

```sh
cd {your assignment root directory} # change directory to the assignment repo
cargo embassy init firmware # create rust project in a subdirectory named "firmware"
```

The resulting file structure should look something like:

{{< filetree/container >}}
  {{< filetree/folder name="assignment-directory" >}}
    {{< filetree/folder name="firmware" >}}
      {{< filetree/folder name=".cargo" state="closed" >}}
          {{< filetree/file name="config.toml" >}}
      {{< /filetree/folder >}}
      {{< filetree/folder name="src" >}}
        {{< filetree/file name="main.rs" >}}
      {{< /filetree/folder >}}
        {{< filetree/file name="Cargo.toml" >}}
        {{< filetree/file name="build.rs" >}}
        {{< filetree/file name="rust-toolchain.toml" >}}
    {{< /filetree/folder >}}
    {{< filetree/file name="README.md" >}}
  {{< /filetree/folder >}}
{{< /filetree/container >}}

{{< callout type="info" >}}
  You can learn more about this structure [here](https://embassy.dev/book/#_project_structure).
{{< /callout >}}

## Checkpoint

Let's make sure all we've done so far is working properly.

In the `firmware` directory, execute:

```sh
cargo build --release
```

> What does `--release` mean? Rust has multiple build profiles for different contexts.
> By default, the `debug` profile is used, this includes debug symbols in the resulting
> binary, and significantly reduces the optimizations done. `release` contains no debug
symbols and is optimized (as specified in `Cargo.toml`).

If this succeeds, plug your DevBoard into your computer and execute:

```sh
cargo run --release
```

> It will ask you to select the port the DevBoard is connected to.

If you see the builtin LED blinking, and `Hello, World!` printing on each blink, you have
successfully deployed Rust code to an ESP32!

Give yourself a pat on the back :)
