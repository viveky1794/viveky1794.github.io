---
layout: home
title: Home
nav_exclude: true
permalink: /
description: "Vivek Yadav -- embedded systems engineer working on QNX, Linux internals, ARM SoC bring-up, and power management. Notes and projects."
---

<div class="scope-page" markdown="1">

<section class="hero" markdown="1">

<div class="hero-status">
<span><span class="status-dot"></span>CH1 ARMED&nbsp;&nbsp;&nbsp;CH2 ARMED&nbsp;&nbsp;&nbsp;T/DIV 100ms</span>
<span>FIG. HOME</span>
</div>

<p class="hero-eyebrow">Embedded Systems Engineer</p>

<h1 class="hero-name"><span class="name-line">Vivek</span><span class="name-line name-accent">Yadav</span></h1>

<svg class="wave-divider" viewBox="0 0 1200 52" preserveAspectRatio="none" aria-hidden="true">
<path d="M 0,46 L 57.8,46 L 72.3,6 L 173.5,6 L 188.0,46 L 216.9,46 L 231.3,6 L 260.2,6 L 274.7,46 L 404.8,46 L 419.3,6 L 621.7,6 L 636.1,46 L 679.5,46 L 694.0,6 L 737.3,6 L 751.8,46 L 795.2,46 L 809.6,6 L 1098.8,6 L 1113.3,46 L 1200.0,46" />
</svg>

<p class="hero-tagline">QNX Platform &middot; Linux Internals &middot; ARM SoC Bring-Up</p>

<p class="hero-bio" markdown="1">
I write low-level software for ARM-based SoCs and microcontrollers, across
both pre- and post-silicon: Linux device drivers, bare-metal firmware,
board support packages, and power management. Currently a Sr. Lead Engineer
on the **QNX Platform team at Qualcomm**, writing wired I/O drivers and
working with hypervisor execution environments. Before that, I spent three
years at **Samsung Semiconductor** on foundry software — bare-metal and
Linux drivers, PMIC/CPUIdle power management, and board bring-up on Cortex-M
and Cortex-A silicon.
</p>

<div class="hero-actions">
<a href="https://www.linkedin.com/in/viveky1794" class="btn-scope btn-scope-primary" target="_blank" rel="noopener">{% include icon-linkedin.html %} Connect on LinkedIn</a>
<a href="/linux-internals" class="btn-scope btn-scope-secondary">Browse the Notes {% include icon-arrow-right.html %}</a>
</div>

</section>

## Skills

<div class="skills-grid" markdown="1">

<div class="skill-group" markdown="1">
**Languages**
C · C++ · Python
</div>

<div class="skill-group" markdown="1">
**Platforms**
ARM Cortex-M · ARM Cortex-A · ESP8266/ESP32 · STM32F1/F4 · 8051 · BeagleBone Black · Raspberry Pi
</div>

<div class="skill-group" markdown="1">
**Focus Areas**
Linux device drivers · Bare-metal firmware · BSP development · Power management (PMIC, CPUIdle, runtime PM) · Cache coherency · Memory management
</div>

<div class="skill-group" markdown="1">
**Protocols**
I2C · USART · SPI · USB (host & device)
</div>

<div class="skill-group" markdown="1">
**Tools**
Git/Gerrit · JTAG (Trace32) · GDB · Buildroot · U-Boot · Yocto · STM32Cube · PyQt5
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
[USB Protocol](/linux-internals/usb-protocol).

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

<div class="project-card notes-card" markdown="1">

### [Linux Internals](/linux-internals)

Computer architecture, memory & virtual memory, caching, process/threading,
scheduling, interrupts, DMA, USB, and kernel boot.

</div>

<div class="project-card notes-card" markdown="1">

### [QNX](/qnx)

Virtualization and hypervisor fundamentals, growing alongside my current
platform work.

</div>

</div>

</div>
