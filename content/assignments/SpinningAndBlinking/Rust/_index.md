---
title: Rust
type: docs
prev: assignments/SpinningAndBlinking/
next: assignments/SpinningAndBlinking/Rust/setup
weight: 2
---

Rust is an esoteric[^1] multiparadigm[^2] systems programming language with an emphasis on safety.
It leverages what we have learned from the past 50 years of advancement in the field of software engineering
to create a consistent, performant, vibrant, reliable, and most importantly *safe* development environment.

When coming from C++, Python, Java, etc. learning Rust can be tricky because you will have to shake the
habit of using anti-patterns[^3] without any consequences. This learning curve can be steep, but over time you
will come to realize that as you endeavor to learn Rust, you *also* become a better developer in other
languages! The ideals Rust holds dear are simply the good design practices of the past half-century all combined
in one programming language.

{{< callout type="info" >}}
  To help you on your journey, refer to the [resources]({{< ref "/additional-resources/rust/#embedded" >}}) page.
{{< /callout >}}

## Some Background

So why exactly is Rust on embedded so cool? What does it provide?
Well, in addition to the language's passive benefits, i.e. static analysis[^4], RAII adherence[^5], fantastic build tools[^6], etc.
Rust also provides us with first class[^7] async[^8]!

You may have heard of RTOS before, it stands for **R**eal **T**ime **O**perating **S**ystem. For complex embedded systems, it can be
really useful to compartmentalize various responsibilities of the system as tasks that run side by side. Of course, the CPU can only
execute one instruction at a time, but by interleaving the instructions, the illusion of parallelism is created. This is called **concurrency**.

In addition to this, microcontrollers are comprised of more than just a CPU, there are many peripherals[^9] the CPU uses to interact with the world.
These peripherals can do work while the CPU is handling other things, and use **D**irect **M**emory **A**ccess (DMA) to read, store, or exchange
information as it is served. A very special Rust crate[^10] called **Embassy** utilizes these very principles to provide a high level async interface
for us to use.

{{< callout type="info" >}}
  More information on Embassy can be found on the [resources]({{< ref "/additional-resources/rust/#embedded" >}}) page.
{{< /callout >}}

[^1]: Languages that push the boundaries of programming with radical new archetypes.
[^2]: Languages that employ multiple programming paradigms, i.e. functional, OOP, etc.
[^3]: Design patterns known for resulting in low-quality code.
[^4]: Programs can be represented with **F**inite **S**tate **M**achines (FSM), control flow can be encoded as types, a strong type system henceforth allows
a program's behavior to be bounded and thus analyzable at compile-time.
[^5]: **R**esource **A**cquisition **I**s **I**nitialization (RAII). We bind resources to symbols, access to the resource
is the same as access to the binding. Now symbolic access control represents resource access control.
[^6]: Cargo -- Rust's build tool -- is renowned for ease of use, performance, and reliability.
[^7]: A first class citizen is a feature a language provides directly.
A third class citizen is a feature *derived* from other primitives of the language.
[^8]: A modern language feature that fascilitates concurrency.
[^9]: The core of microcontrollers is the concept of peripherals. The other "entities" that coexist with the CPU.
UART, Timers, ADC, etc.
[^10]: *Crate* is the term used for Rust packages. But *crate* is not a synonym
for *library*. Libraries are distributed as precompiled binaries, crates are
distributed as source code. This means when you build a Rust program, you also
build all dependencies from source.
