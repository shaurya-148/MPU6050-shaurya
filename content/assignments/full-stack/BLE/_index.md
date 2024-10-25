---
title: BLE
type: docs
prev: assignments/full-stack/
next: assignments/full-stack/ble/firmware
weight: 2
---

Let's take a look at the hierarchy of the system we are about to design:

```mermaid
graph LR
    subgraph "DevBoard (Rust)"
        subgraph "Front End"
            A2(LED)
        end

        subgraph "Back End"
            B2(BLE)
            C2(Events)
            D2(Logic)
        end

        B2 --> C2 --> D2 --> A2
        D2 --> B2
    end

    subgraph "Phone (Swift)"
        subgraph "Front End"
            A1(UI)
            B1(Alerts)
        end

        subgraph "Back End"
            C1(Delegate)
            D1(Logic)
            E1(BLE)
        end

        A1 --> C1 --> D1 <--> E1
        D1 --> B1
    end

    B2 <--> E1
```

Now, I know this looks complicated, but we will tackle each block one by one, nice and slow.

Once you are done with this, you will have a solid understanding of creating systems involving
bidirectional communication between 2 devices, which is *super* useful for *a ton* of
applications (including your final project).

So let's get started.

## How Does BLE Work?

Modern BLE (Bluetooth Low Energy) systems rely on the *Service -> Characteristic* architecture.

Your iPhone can scan for BLE devices advertising **services**, to decide whether to connect to them or not.

A service is like an Object, it contains states, and represents some kind of data or behavior.

Services contain **characteristics**, which are the states of the service. You can configure characteristics to be readable and writable.

In this case, we will have one service with one characteristic: *LED*.

Here is a flowchart of how our BLE system will work:

```mermaid
flowchart TD
P(Peripheral) --> L
    subgraph "Client (iPhone)"
    C(Central Manager) --> P
    end

    subgraph "Server (ESP32)"
    subgraph Services
    L(MyFirstService) --> R
    L --> G
    L --> B
    subgraph Characteristics
    R(C0)
    G(C1)
    B(C2)
    end
    end
    end
```

This layout is called **GATT** (Generic ATTribute Profile)
