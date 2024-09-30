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
