---
title: Battery Indicator with ESP32
date: 2024-11-21
authors:
  - name: Andy Tu
  - name: Madeline Fruin
  - name: William Luong
---

![Battery](Team13Photos/BatteryCharged.jpg) 
## Introduction

This tutorial will teach readers how to use ESP32's ADC pins to measure a battery's charge. Since the ADC pins can take in a max of 3.3V, we will be covering how to implement a voltage divider to lower the maximum voltage of a battery to an amount allowed by the ESP32. For the coding section, we will use the Arduino IDE to write code that will read, display, and convert the battery readings to a charge percentage using a function.

### Learning Objectives

- Voltage Dividers
- ESP32 ADC/GPIO Pins
- Arduino C
- Battery Charge State

### Background Information

A battery indicator circuit measures the state of charge (SoC) of a battery and visually outputs whether the battery’s capacity is low or full. These circuits are commonly used in battery management systems, smartphones, and other consumer electronics.

Pros:
1. Provides real-time feedback on battery life.
2. Prevents damage to the battery by avoiding overcharging or over-discharging.
3. Offers insights to maximize battery life and performance.

Cons:

1. The more complex the circuit, the higher the cost (e.g., precise resistors may be required).
2. Limited accuracy, as voltage alone isn’t the only factor that determines the charge state.

Key Concepts:

1. Voltage Divider
2. Resistor Ratios with Multiple Batteries
3. Arduino Basics

## Getting Started

You will receive an ESP32 board and USB-C to USB-C connector cable. The ESP32 board utilizes the ESP32 (obviously). The ESP32 is a system on a chip with Wifi and bluetooth capacbilities. This means that it can be used to make IoT projects(Internet of things). However, we will be using it very simply, you do not need to understand everything it can do for this tutorial.

You will also receive a breadboard, jumper cables, and resistors. The rows of the breadboard are connected while the columns are not. The resistors and jumper cables should fit snuggly into the holes of the breadboard. 

We will be using the Arduino IDE (Integrated Development Environment). This is a free, open source program that allows users to write code and upload it to boards. 

### Required Downloads and Installations

If you don't have the Arduino IDE already, download it [here](https://www.arduino.cc/en/software).

### Required Components

List your required hardware components and the quantities here.

| Component Name | Quanitity |
| -------------- | --------- |
|     ESP32      |     1     |
| ESP32 Connector Cable | 1  |
|Resistors (47kΩ)|     2     |
| 18650 Battery  |     1     |
| Battery Holder |     1     |
|Mini Breadboard |     1     |
| Jumper Cables  |     2     |


### Required Tools and Equipment

Computer, Arduino IDE

## Part 01: Setting up the Circuit

### Introduction

In this section, we will be discussing the voltage divider and circuit setup. We will not be writing any code, so a computer is not yet needed.

### Objective

- Understand how to pick values for a voltage divider.
- Understand how to properly connect the battery, resistors, and ESP32.

### Background Information

Give a brief explanation of the technical skills learned/needed
in this challenge. There is no need to go into detail as a
separation document should be prepared to explain more in depth
about the technical skills

If you've taken a look at the battery we gave you, you might notice that it says 3.7V on it. The maximum voltage input for the ESP32 is 3.3V, so we need to find a way to decrease the voltage from the battery, we can do this with a <b>voltage divider</b>.

A voltage divider is a passive linear circuit made up of two resistors, although sometimes more are used to get a specific resistance value. 
![Voltage Divider Schematic and Equation](Team13Photos/voltagedivider.png)

We have given you two 47kΩ resistors, so the output voltage will be 3.7V * 47kΩ/(47kΩ + 47kΩ) = 1.85V. 

Notice that our voltage divider halves the input voltage. To combat this division, we will need to remember to multiply our measured voltage by 2 (the reciprocal).

### Components

- Breadboard
- Battery and Battery Holder
- Resistors
- ESP32
- Jumper Cables

### Instructional

Assemble the circuit as shown in the below image. 
![Circuit Fritz](Team13Photos/Fritz.png)
Make sure to attach the cathode of the battery to the voltage divider *AND* to the ground pin of the ESP32!

## Part 02: Writing the Code

### Introduction

In this section, we will be discussing the code used to measure the battery's percentage. 

### Objective

- Understand the Arduino interface.
- Understand how to assign pin numbers in Arduino.
- Understand how to implement functions in Arduino.

### Background Information

This section focuses on coding in the Arduino IDE. The language is very similar to C or C++. If you have any background knowledge in those languages, you will recognize the code and its formatting. If you don't have experience in C/C++, don't worry, we will be breaking down the code into very manageable chunks. 
*Hint: If you don't want to rewrite all the code, use the copy button in the upper right corner of the code blocks!*

### Components

- ESP32 (with the circuit you just built attached)
- USB-C to USB-C Connector cable for ESP32
- Your Computer

### Instructional

Another goal of the project is to integrate both hardware and software to create a simple battery indicator system using the ESP32 microcontroller.
Let's take a dive into the indvidual blocks of code. 

```C
{
  const int batteryPin = 1;
  const float referenceVoltage = 3.3;
  const int resolution = 3950;
  const float voltageDividerRatio = 2.0;
  const float maxBatt = 4.2;
  const float minBatt = 3;
}
```
- `batteryPin` is the analog pin connected to the voltage divider's output
- `referenceVoltage` is the maximum voltage the ADC can read. (normally it's 3.3V for the ESP32)
- `resolution` defines the ADC's bit resolution, which determines how anlog voltage is assigned to digital values
- `voltageDividerRatio` the scaling factor of the voltage divider
- `maxBatt` and `minBatt` defines the battery's voltage range. 

```C
{
  float readBatteryVoltage() {
    int rawADC = analogRead(?)
    float voltageAtPin = ?
    float batteryVoltage = ? 
    return batteryVoltage;
  }
}
```

Fill out the code function that calculates the battery voltage.
- `analogRead` captures the raw ADC value corresponding to the voltage at the ESP32 pin.
- `voltageAtPin` using the provided code and understanding of the ADC conversion, write the line of code that calculates it.
  - The ADC reading is a fraction of the max resolution.
  - mulitply this fraction by the reference voltage.
- `batteryVoltage` After finding the voltageAtPin how can we find the actual battery voltage (Hint: remember the scaling factor).

```C
{
  void setup() {
    Serial.begin(115200);
    pinMode(?);
  }
}
```

- `Serial.begin(115200)` sets up communication with the serial monitor for real-time data output.
- `pinMode` This should configures the ADC pin as an input to read the voltage of the circuit.

```C
{
  void loop() {
    float batteryVoltage = readBatteryVoltage();
    Serial.print("Battery Voltage: ");
    Serial.print(batteryVoltage);
    Serial.println(" V");

    int battPercent = ((batteryVoltage-minBatt)/(maxBatt-minBatt))*100;
    Serial.print("Battery Percentage: ");
    Serial.print(battPercent);
    Serial.println(" %");

    delay(10000);
  }
}
```
- The loop continously monitors and outputs the battery's state:
- `readBatteryVoltage()` gets the current battery voltage
- `Serial.print` displays the voltage in the serial monitor
- `battPercent` calculates the battery's percentage
- `delay(10000)` pauses the loop for 10 seconds

Overall this function should provide us with real-time battery status.
## Final Circuit Image


![Photo of Real-Life Completed Circuit](Team13Photos/Final_circuit_Image.jpg)

Circuit setup should look similar to the image above.

### Arduino Ouput

![Sample Output](Team13Photos/Arduino_output.png)

Your Arduino output should print this several times (your voltage percentage may differ depending on battery).
### Analysis

The example uses an ESP32 and a voltage divider to measure a battery's state of charge. This circuit demostrates the core concepts like ADC functionality and Arduino coding. The voltage divder lowers the battery voltage(3.7V to 1.85V) to a safe level for the ESP32 to read. Once the circuit is connected on the breadboard and linked to the ESP32, the Arduino code will adjust for the voltage scaling to calculate and provide an ouput of both the actual voltage and charge percentage. You can see how this employs the concepts of circuit design and programming with the readBatteryVoltage function by converting ADC readings into functional voltage data. The serial monitor should output real-time voltage and it helps validate the system's accuracy. 

## Challenge Questions

How would this change if we were using a 4.2V 18650 battery? Would you need to change either of the values of the voltage divider or just the code? 

What about a 12V battery?

### Challenge Answers

We would not need to change the voltage divider for a 4.2V battery because 4.2V/2 is 2.1V, which is under the maximum that the board can take in. We would only need to change the minBatt and maxBatt values in the code.

We would need to change the voltage divider for a 12V battery because 12V/2 is 6V, which is nearly double what the board can take in! We could choose R1 = 300kΩ and R2 = 100kΩ to get 1/4 of 12V, or 3V. We would have to change our maxBatt, minBatt, *and VoltageDividerRatio* in the code.

## Additional Resources

### Useful links

[Website that tells the nominal voltage and voltage range of many batteries.](https://www.monolithicpower.com/learning/resources/an-introduction-to-batteries-components-parameters-types-and-chargers)
