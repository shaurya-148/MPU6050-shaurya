---
title: Full Stack
type: docs
prev: assignments/spinning-and-blinking/
next: assignments/full-stack/tutorial
weight: 5
---

## Preamble

You may have heard the term "full stack" before. Web developers have coined it as meaning all the systems from the database (backend) to the user interface (frontend).

But web developers don't get all the fun. As Electrical Engineers, we have our *own* definition of "full stack".

For us, the full stack is:

1. Hardware
1. Firmware
1. Communication
1. User Interface

You're halfway there! In one fell swoop, we're going to finish off the last two.

## Assignment

In this assignment you will become a full stack engineer by learning to create a **G**raphical **U**ser **I**nterface (GUI) and a communication
system to interact with a microcontroller.

You will create a **client** (the GUI running on your computer/phone) which communicates with a **peripheral** (your microcontroller) over some medium.

**You are once again faced with an important decision**

You have **two** pathways to choose from:

{{< cards >}}
  {{< card link="usb" title="USB: Python & Arduino" >}}
  {{< card link="ble" title="BLE: Swift & Rust" >}}
{{< /cards >}}

{{< callout type="warning" >}}
  You **must** have a Mac and an iPhone to use Swift for this assignment.
{{< /callout >}}

{{< callout type="warning" >}}
  Only use Rust if you completed the Rust path from [Spinning and Blinking]({{< ref "/assignments/spinning-and-blinking/rust" >}}).
{{< /callout >}}

## Your Options
### USB

This option will teach you how to conduct serial communication over USB[^1].
On your computer you will use **Python** to create the client.

### BLE

This option will teach you how to communicate with BLE[^2]. You will develop an app
for your phone using **Swift** and **SwiftUI**.

Swift is a programming language made by Apple[^3] primarily for app development. SwiftUI is
the framework Apple provides for declaratively designing modern, responsive, and frankly
beautiful user interfaces across the Apple ecosystem.

[^1]: **U**niversal **S**erial **B**us (USB). The classic ports on your computer, phone, and the DevBoard.
[^2]: **B**luetooth **L**ow **E**nergy (BLE). The most modern "style" of Bluetooth communication.
[^3]: Chris Latner was an employee at Apple when he created Swift. He received a lot of push back,
and in fact, had to keep it mostly a secret until he had enough notoriety at the company to
convince his higher ups to devote resources to it.
He also created LLVM, the world's most advanced compiler backend, which Rust (and naturally Swift) **relies on** to provide many of it's
features. This one man has improved the lives of tens, if not hundreds of millions of people simply
by following his passion for engineering and design, despite the adversity he faced.
He has ushered in the next chapter of global technological advancement. It would be wise
to read about his life, you could very quickly become a better engineer!
