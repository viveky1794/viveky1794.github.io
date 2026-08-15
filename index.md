---
layout: home
title: Home
nav_exclude: true
permalink: /
description: "Vivek Yadav -- embedded systems engineer working on QNX, Linux internals, ARM SoC bring-up, and power management. Notes and projects."
---

# Vivek Yadav

**Embedded Systems Engineer — QNX, Linux & ARM · 9 years**

I write low-level software for ARM-based SoCs and microcontrollers, across
both pre- and post-silicon: Linux device drivers, bare-metal firmware,
board support packages, and power management. Currently a Sr. Lead Engineer
on the **QNX Platform team at Qualcomm**, writing wired I/O drivers and
working with hypervisor execution environments. Before that, I spent three
years at **Samsung Semiconductor** on foundry software — bare-metal and
Linux drivers, PMIC/CPUIdle power management, and board bring-up on Cortex-M
and Cortex-A silicon.

This site is where I write up what I'm learning as I go — partly so it
sticks, partly because it's the kind of reference I'd have wanted myself.

[Connect on LinkedIn](https://www.linkedin.com/in/viveky1794){: .btn .btn-primary }
[Browse the Notes](/linux-internals){: .btn }

## Skills

**Languages:** C, C++, Python
**Platforms:** ARM Cortex-M, ARM Cortex-A, ESP8266/ESP32, STM32F1/F4, 8051, BeagleBone Black, Raspberry Pi
**Focus areas:** Linux device drivers, bare-metal firmware, BSP development, power management (PMIC, CPUIdle, runtime PM), cache coherency, memory management
**Protocols:** I2C, USART, SPI, USB (host & device)
**Tools:** Git/Gerrit, JTAG (Trace32), GDB, Buildroot, U-Boot, Yocto, STM32Cube, PyQt5

## Projects

Things I've built, beyond the write-ups:

### Custom USB Host/HID Stack (STM32F407)
A from-scratch USB Host driver and printer class on the STM32 HAL, receiving
data over USART and forwarding it across USB to drive generic USB printers.
Extended into a full USB HID implementation supporting three device
classes — barcode scanner, receipt printer, and a custom keyboard — for a
point-of-sale hardware integration. Background reading:
[USB Protocol](/linux-internals/usb-protocol).
{: .d-inline-block }
`C` `STM32 HAL` `USB Host` `HID`
{: .label .label-blue }

### PMIC Control GUI
A PyQt5 desktop tool for controlling and monitoring power delivery over
PMBus/I2C during ASIC bring-up — built to streamline the SoC design team's
workflow and give RTL/PV engineers a tailored way to exercise the power
management hardware without hand-written I2C scripts.
{: .d-inline-block }
`Python` `PyQt5` `I2C / PMBus` `Power Management`
{: .label .label-blue }

### OpenCV Warehouse Item Locator
An OpenCV application driving a standard projector to highlight the correct
box in large warehouse racks, cutting item-retrieval time and error rate for
warehouse operators by projecting directly onto the physical shelf instead
of relying on a handheld screen.
{: .d-inline-block }
`Python` `OpenCV` `Computer Vision`
{: .label .label-blue }

## Notes

A working knowledge base of Linux and embedded-systems internals, written
as I dig into each topic — not a tutorial series, more a set of references
I keep coming back to.

- **[Linux Internals](/linux-internals)** — computer architecture, memory &
  virtual memory, caching, process/threading, scheduling, interrupts, DMA,
  USB, and kernel boot.
- **[QNX](/qnx)** — virtualization and hypervisor fundamentals, growing
  alongside my current platform work.
