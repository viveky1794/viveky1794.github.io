---
layout: home
title: Home
nav_order: 1
permalink: /
description: "Vivek Yadav -- embedded systems engineer working on QNX, Linux internals, ARM SoC bring-up, and power management. Notes and projects."
---

<div class="hero-row" markdown="1">

<div class="hero-text" markdown="1">

# Vivek Yadav
{: .fs-9 }

Embedded Systems Engineer — QNX, Linux & ARM · 9 years
{: .fs-6 .fw-300 }

I write low-level software for ARM-based SoCs and microcontrollers, across
both pre- and post-silicon: Linux device drivers, bare-metal firmware,
board support packages, and power management. Currently a Sr. Lead Engineer
on the **QNX Platform team at Qualcomm**, writing wired I/O drivers and
working with hypervisor execution environments. Before that, I spent three
years at **Samsung Semiconductor** on foundry software — bare-metal and
Linux drivers, PMIC/CPUIdle power management, and board bring-up on Cortex-M
and Cortex-A silicon.
{: .fs-5 }

[Connect on LinkedIn](https://www.linkedin.com/in/viveky1794){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 target="_blank" rel="noopener" }
[Browse the Notes](/articles/linux/){: .btn .fs-5 .mb-4 .mb-md-0 }

</div>

<img src="/assets/images/vivek.jpg" alt="Photo of Vivek Yadav" class="hero-photo">

</div>

---

## Skills

<div class="skills-grid" markdown="1">

<div class="skill-group" markdown="1">
**Languages**

- C
- C++
- Python
</div>

<div class="skill-group" markdown="1">
**Platforms**

- ARM Cortex-M
- ARM Cortex-A
- ESP8266 / ESP32
- STM32F1 / F4
- 8051
- BeagleBone Black
- Raspberry Pi
</div>

<div class="skill-group" markdown="1">
**Focus Areas**

- Linux device drivers
- Bare-metal firmware
- BSP development
- Power management (PMIC, CPUIdle, runtime PM)
- Cache coherency
- Memory management
</div>

<div class="skill-group" markdown="1">
**Protocols**

- I2C
- USART
- SPI
- USB (host & device)
</div>

<div class="skill-group" markdown="1">
**Tools**

- Git/Gerrit
- JTAG (Trace32)
- GDB
- Buildroot
- U-Boot
- Yocto
- STM32Cube
- PyQt5
</div>

</div>

## Projects

Things I've built, beyond the write-ups.

<div class="project-grid" markdown="1">

<div class="project-card" markdown="1">

### Custom USB Host/HID Stack

<p class="project-eyebrow">STM32F407</p>

A from-scratch USB Host driver and printer class on the STM32 HAL, receiving
data over USART and forwarding it across USB to drive generic USB printers.
Extended into a full USB HID implementation supporting three device
classes — barcode scanner, receipt printer, and a custom keyboard — for a
point-of-sale hardware integration. Background reading:
[USB Protocol](/articles/linux/usb-protocol).

<div class="tag-row">
<span class="tag">C</span><span class="tag">STM32 HAL</span><span class="tag">USB Host</span><span class="tag">HID</span>
</div>

</div>

<div class="project-card" markdown="1">

### PMIC Control GUI

<p class="project-eyebrow">Bring-Up Tooling</p>

A PyQt5 desktop tool for controlling and monitoring power delivery over
PMBus/I2C during ASIC bring-up — built to streamline the SoC design team's
workflow and give RTL/PV engineers a tailored way to exercise the power
management hardware without hand-written I2C scripts.

<div class="tag-row">
<span class="tag">Python</span><span class="tag">PyQt5</span><span class="tag">I2C / PMBus</span><span class="tag">Power Management</span>
</div>

</div>

<div class="project-card" markdown="1">

### OpenCV Warehouse Item Locator

<p class="project-eyebrow">Computer Vision</p>

An OpenCV application driving a standard projector to highlight the correct
box in large warehouse racks, cutting item-retrieval time and error rate for
warehouse operators by projecting directly onto the physical shelf instead
of relying on a handheld screen.

<div class="tag-row">
<span class="tag">Python</span><span class="tag">OpenCV</span><span class="tag">Computer Vision</span>
</div>

</div>

</div>

## Notes

A working knowledge base of Linux and embedded-systems internals, written
as I dig into each topic — not a tutorial series, more a set of references
I keep coming back to.

<div class="project-grid" markdown="1">

<div class="project-card" markdown="1">

### [Linux Internals](/articles/linux/)

Computer architecture, memory & virtual memory, caching, process/threading,
scheduling, interrupts, DMA, USB, and kernel boot.

</div>

<div class="project-card" markdown="1">

### [QNX](/articles/qnx/)

Virtualization and hypervisor fundamentals, growing alongside my current
platform work.

</div>

</div>
