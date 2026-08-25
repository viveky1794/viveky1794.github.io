---
layout: default
title: QNX
parent: Articles
has_children: true
has_toc: false
nav_order: 2
permalink: /articles/qnx/
description: "Virtualization fundamentals -- VMs, hypervisors, Type-1 vs Type-2 -- as background for QNX platform and hypervisor work."
redirect_from:
  - /qnx/
  - /qnx.html
---

# QNX

Notes on virtualization fundamentals, written while ramping up on QNX
platform and hypervisor execution environments in my current role at
Qualcomm. This section will grow as that work does.

## What Is Virtualization?

Virtualization in computer science often refers to the abstraction of some
physical component into a logical object.

## What Is a Virtual Machine?

A virtual machine (VM) can virtualize all of the hardware resources,
including processors, memory, storage, and network connectivity.

## What Is a Hypervisor?

A hypervisor is the software that provides the environment in which VMs
operate.

- Just as an operating system schedules threads from different processes, a
  hypervisor schedules requests from different (or the same) guest OS with
  minimal latency.
- Just as an operating system provides isolation between threads, a
  hypervisor provides isolation between the VMs it hosts.

### Type-1 (Bare-Metal)

Sits directly on the hardware, with no host OS underneath it.

### Type-2 (Hosted)

Sits on top of a host operating system, which in turn talks to the hardware.


## Further Reading

Reference material used while writing these notes (kept in the repo under
`articles/qnx/docs/`):

- *Virtualization Essentials*, Matthew Portnoy (2nd Edition)
- ["What Is a Hypervisor and How Does It Work?"](/articles/qnx/docs/what-is-a-hypervisor-and-how-does-it-work-pt1.pdf)
