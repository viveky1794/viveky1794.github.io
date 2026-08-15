---
layout: default
title: DMA
parent: Linux Internals
nav_order: 7
description: "Direct Memory Access: why it exists, transfer modes, IOMMU, and how an ARM-style DMA controller (PL330) executes its own instruction stream."
---

# Direct Memory Access (DMA)

DMA lets peripherals move data to and from main memory without the CPU
shuffling every byte itself.

## Part 1 — Why DMA Exists

**Key questions this section answers:**

1) What is DMA? Why do we need it?
   **Definition:** DMA is a mechanism that allows data transfer between memory and peripherals without CPU intervention, improving overall system efficiency.
2) What is 1st party and 3rd party DMA?
3) What are the operating modes of DMA?
4) What is IOMMU? Why do we need it?

---

If I ask a random person on the street what the parts of a computer are, they would probably mention things like the monitor, mouse, keyboard, or hard drive.
They might also mention the CPU, RAM, or a graphics card if they're a little more tech-savvy or play computer games. However, at the most fundamental level,
a basic computer only needs three components: a central processing unit, memory, and a bus to connect and manage data transfers between them.
A power supply is also necessary, but here I'm focusing on the functional aspects of a computer. All other components, such as USB devices, storage drives,
network adapters, audio hardware, and graphics cards, are considered peripheral devices.

A typical computer includes many peripheral devices, most of which need to read or write data to main memory from time to time, some more than others.
For example, running a video game requires a constant flow of data between RAM and the graphics card's internal memory, called VRAM. This makes main
memory a shared resource for the CPU and all other connected devices.

![1749701972686](/Linux/Core/image/Linux_DMA/1749701972686.png)

**Transferring data between main memory and the peripheral device is not direct, as I'm showing here; it requires executing several instructions, such as reading data from RAM and writing it to the device's internal memory. These operations can only be performed by a unit capable of processing such instructions, like the CPU. But if the CPU had to manually move every piece of data that a device needs, it would spend most of its time shuffling data back and forth instead of doing what it's supposed to do: running programs.**

![1749702118390](/Linux/Core/image/Linux_DMA/1749702118390.png)

**To solve this problem, computers include a specialized processor called a Direct Memory Access (DMA) Controller, which handles data transfers for peripheral devices independently of the CPU. This allows the CPU to focus on executing program instructions without being constantly interrupted by data transfer requests.**

![1749702196055](/Linux/Core/image/Linux_DMA/1749702196055.png)

**Older systems that used PCI or ISA bus architectures relied on a shared bus that connected the CPU, main memory, and peripheral devices. At any given moment, only one entity could control the bus.** If multiple entities attempted to drive the bus lines at the same time, a condition known as bus **contention would occur, leading to delays or data corruption**.

![1749702273198](/Linux/Core/image/Linux_DMA/1749702273198.png)

To prevent this, a centralized hardware component called the **bus arbiter managed access to the bus**, ensuring that only one processor or device had control at any given moment.

![1749702305148](/Linux/Core/image/Linux_DMA/1749702305148.png)

**In those older computer architectures, a single DMA controller was responsible for transferring data between peripheral devices and main memory. This configuration is known as a centralized DMA controller or third-party DMA. When a device needed to transfer data, it would send a request to the DMA controller. The DMA controller, in turn, would request control of the bus from the bus arbiter. Once bus control was granted, the DMA controller would perform the actual data transfer between the device and main memory. If multiple devices requested data transfer simultaneously, the DMA controller would prioritize them according to a specific arbitration algorithm.**

![1749702414393](/Linux/Core/image/Linux_DMA/1749702414393.png)

**While this system worked, it eventually became a bottleneck because the centralized DMA controller couldn't keep up with the increasing speed and throughput demands of newer generations of computers.**

**To overcome this limitation, data transfers no longer relied on a centralized DMA controller. Instead, each peripheral device was designed with its own internal DMA controller, allowing it to independently manage data transfer. This technique is known as bus mastering or first-party DMA**. With this approach, devices could access their internal memory faster, request control of the bus, and transfer data without depending on a shared DMA controller. However, **if multiple devices needed to use the bus simultaneously, the bus arbiter was responsible for managing access. Since the CPU is typically the most critical component in a system, it was often given higher priority over other devices when requesting bus access.**

![1749702545004](/Linux/Core/image/Linux_DMA/1749702545004.png)

**DMA controllers transfer data in a way that allows the device to operate efficiently while trying to maximize CPU access to the bus. They achieve this by alternating between three transfer modes:**

**1) Burst mode,
2) Cycle stealing mode,
3) Transparent mode.**

![1749702605470](/Linux/Core/image/Linux_DMA/1749702605470.png)

**Burst mode** is used when large chunks of data must be quickly moved between a peripheral device and main memory. In this mode, the DMA controller gains full control over the memory bus, which may temporarily block the processor from accessing the bus while the transfer occurs. Once the transfer is complete, the DMA controller releases the bus, allowing the bus arbiter to grant access to the next waiting processor or device.

![1749702630029](/Linux/Core/image/Linux_DMA/1749702630029.png)

**Cycle stealing mode** is used by devices that frequently transfer small amounts of data, such as network cards or sound cards. Here, the DMA controller takes control of the bus for just a few cycles at a time, allowing the CPU to continue execution with minimal interruptions. While this mode does introduce minor slowdowns, it is far less disruptive than burst mode, which locks the bus for an extended period.

![1749702687309](/Linux/Core/image/Linux_DMA/1749702687309.png)

**Transparent mode** is the **most CPU-friendly**. **In this mode, the DMA controller only transfers data when the memory bus is completely idle, meaning no other component is using it. Since the CPU is the primary user of the bus,** this ensures the DMA transfers never interfere with its memory access requests. The trade-off is that while this mode eliminates CPU slowdowns, **it can result in very slow DMA transfers if the bus is frequently occupied.**

![1749702754881](/Linux/Core/image/Linux_DMA/1749702754881.png)

![1749702768559](/Linux/Core/image/Linux_DMA/1749702768559.png)

![1749702778610](/Linux/Core/image/Linux_DMA/1749702778610.png)

***The good news is that regardless of the transfer mode used by a DMA controller, the CPU can often continue executing instructions even in burst mode. This is thanks to the processor's caching system, which allows it to fetch frequently used data from cache instead of accessing main memory. In most cases, the CPU can continue running instructions using cached data. However, if a cache miss occurs, meaning the required data isn't in the cache, the processor must request data from main memory. If a DMA controller is currently using the memory bus, the CPU may experience a stall while waiting for the bus arbiter to grant access**.*

![1749704487559](/Linux/Core/image/Linux_DMA/1749704487559.png)

Today, most general-purpose computers use PCI Express as the main expansion bus standard. Unlike older PCI-based systems that relied on a single shared bus, **PCI Express uses a point-to-point topology where data is routed between the CPU, memory, and peripheral devices through switches that can manage multiple data transfers happening at the same time.** **In a PCI Express system, each peripheral device must have its own DMA controller to handle data transfers, similar to bus mastering in older architectures. However, unlike traditional bus mastering, where devices had to compete for control of a shared bus, PCI Express allows multiple devices to send and receive data simultaneously using their own dedicated lanes. The PCI Express switches and root complex dynamically manage the flow of information to ensure efficient data transfers and prevent bus contention.**

![1749704605369](/Linux/Core/image/Linux_DMA/1749704605369.png)

![1749704627174](/Linux/Core/image/Linux_DMA/1749704627174.png)

If you're familiar with virtual memory, you might notice a potential issue here. DMA controllers are processors designed to transfer data from point A to point B; they aren't aware of the current state of main memory, nor do they need to be. So how do they know where to write or read data from in physical memory? Just like any running process, peripheral devices operate in a virtual address space. This gives them the illusion of having exclusive access to the entire memory space. **Behind the scenes, the operating system maps a device's virtual addresses to physical memory using a mechanism known as I/O page tables.** To accelerate this translation process, a specialized hardware component called the **input/output memory management unit is used**. Every time a device initiates a DMA transfer, its DMA controller must go through the unit to translate virtual addresses into physical ones. This not only enables fast address translation but also enforces memory protection, preventing devices from accessing memory they don't own and helping mitigate DMA-related cyber attacks.

![1749704870108](/Linux/Core/image/Linux_DMA/1749704870108.png)

![1749704841758](/Linux/Core/image/Linux_DMA/1749704841758.png)

## Part 2 — Inside an ARM DMA Controller (PL330-style)

### Direct Memory Access Controller

1. Read **DMA330 ARM official** document.

### What is Direct memory access controller i.e DMAC ?

1) The DMAC is an Advanced Microcontroller Bus Architecture (AMBA) compliant peripheral
   that is developed, tested, and licensed by ARM.
2) The DMAC provides an AXI master interface to perform the DMA transfers and two APB slave
   interfaces that control its operation.
3) The DMAC includes a small instruction set that provides a flexible method of specifying the
   DMA operations.

![1753681826746](/Linux/Core/image/Linux_DMA_part-2/1753681826746.png)

### Basic Terminolgy

1) **DMA channel =>** A section of the DMAC that controls a DMA cycle by executing its own
   program thread. You can configure the number of channels that the DMAC contains.
2) **DMA cycle =>** All the DMA transfers that the DMAC must perform, to transfer the
   programmed number of data packets.
3) **DMA manager =>** A section of the DMAC that manages the operation of the DMAC by
   executing its own program thread.
4) **DMA transfer =>** The action of transferring a single byte, halfword, or word.

NOTE:

```
In the context of DMA (Direct Memory Access), a thread here does not refer to:
* a CPU software thread (like in POSIX threads or an OS task), or

* a CPU hardware thread (like a Hyper-Threading core).

Instead, it refers to a simple, independent control sequence inside the DMA controller.
It is best described as a **microcoded or state-machine thread — a hardware-executed finite-state machine programmed** to:

* fetch DMA descriptors (transfer parameters),

* follow a sequence (load, wait, store, trigger interrupt),

and run independently of the CPU.

📦 Analogy
Imagine the DMAC as a factory, and each DMA channel is a robot arm with its own scripted job (its "thread"):

Channel 0: Move data from memory A to memory B.

Channel 1: Move audio data to I2S buffer.

Each robot works independently, following its own instructions.
```

### Overview

![1753682494608](/Linux/Core/image/Linux_DMA_part-2/1753682494608.png)

The DMAC contains an instruction processing block that enables it to process program code that
controls a DMA transfer.

`DMAC has logic unit which is able to decode some set of instructions specifically meant for DMA operations`

The program code is stored in a region of system memory that the DMAC accesses using its AXI master interface.

`DMA program code take place in .data section i.e RAM not in .text section of program binary. Detail step by step approach is given in below table.`

The DMAC stores instructions temporarily in a cache. You can configure the line length and depth of the cache.

| Step | Who Does It                        | What Happens                                                                                                                                 |
| ---- | ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.   | **CPU**                      | Allocates a memory buffer (in RAM) to hold**DMA instructions** (e.g., `DMAMOV`, `DMAST`, `DMAEND`)                               |
| 2.   | **CPU**                      | Fills that buffer with  **32-bit opcodes**, just like writing normal data                                                            |
| 3.   | **CPU**                      | Configures the DMA controller (via MMIO) — including passing the**start address** of that buffer into the **`EXEC` register** |
| 4.   | **DMA Controller** (not CPU) | Starts fetching instructions from the address provided (via AXI bus)                                                                         |
| 5.   | **DMA Controller**           | Decodes and executes them internally: reads data from source, writes to destination, loops, raises IRQs, etc.                                |
| 6.   | **DMA Controller**           | Stops when it hits a `DMAEND` instruction                                                                                                  |

🔧 Think of the DMA Instruction Stream Like a Script

* It's not compiled code
* It’s a binary script for the DMA engine’s interpreter
* It’s stored like an array in RAM
* The CPU only needs to prepare it and point to it

This Makes DMA Programmable

* Reuse the same DMA hardware for many different patterns (memcpy, scatter-gather, looping, conditional)
* Offload memory movement work from the CPU
* Execute these transfers independently and in parallel

**The DMA Manager Thread is not tied to data transfers.**

Instead, it’s like the “orchestrator” that:

* Executes setup/housekeeping instructions
* Starts/stops channel threads
* Handles instructions like DMASEV, DMAKILL, DMAFLUSHP
* Its instruction stream is separate and lives in a dedicated buffer

**The DMA controller has a small instruction execution engine that:**

* `Can fetch and execute one 32-bit instruction per AXI clock`
* This includes channel threads and manager thread instructions

"To ensure that it regularly executes each active thread, it alternates by processing the DMA manager thread and then a DMA channel thread."

**The controller uses a round-robin scheduler:**

* Execute one manager thread instruction
* Execute one channel thread instruction
* Repeat...

This interleaving ensures:

* The manager thread doesn’t get starved
* All active channel threads make progress

`It provides a separate Program Counter (PC) register for each DMA channel.`
`The DMAC also contains a Multi First-In-First-Out (MFIFO) data buffer that it uses to store data that it reads, or writes, during a DMA transfer.`

### Interview QnA

Here are 50 DMA (Direct Memory Access) interview or self-study questions arranged from beginner to expert level, focusing on embedded systems:

---

### Beginner Level (1–15)

Basic understanding and DMA fundamentals.

1. What is DMA and why is it used in embedded systems?

    DMA (Direct Memory Access) is a hardware feature that allows peripherals or devices to transfer data directly to or from system memory without involving the CPU for each data transaction.

    In simple terms, the CPU programs the DMA controller once, and then the DMA hardware becomes a mini-processor that moves data between memory and peripherals automatically

2. How does DMA differ from CPU-driven data transfer?

3. What are the basic components of a DMA controller?
    1)

4. What are typical peripherals that use DMA?
5. What are the common transfer types in DMA (e.g., memory-to-memory, peripheral-to-memory)?
6. What is the role of the DMA request (DREQ) and acknowledge (DACK) signals?
7. What are burst mode and single mode in DMA transfers?
8. What is a DMA channel?
9. How does DMA improve CPU performance?
10. What is meant by DMA latency?
11. What happens when DMA and CPU access the same memory at the same time?
12. What is the difference between polling, interrupt-driven, and DMA-based data transfer?
13. Can a DMA controller be programmed by software?
14. What is scatter-gather in DMA?
15. How is DMA configured in an embedded system like STM32 or LPC?

---

### Intermediate Level (16–35)

Understanding DMA registers, integration, and use in real-time systems.

16. How do you configure a DMA transfer in STM32 (or your platform).
17. What are circular and linear modes in DMA.
18. How do you link DMA with UART or SPI in practice.
19. How does a DMA interrupt handler work.
20. What is a transfer complete (TC) interrupt in DMA.
21. How does DMA handle priority between multiple channels.
22. What is double buffering in DMA.
23. How do you avoid cache coherency issues with DMA and CPU sharing memory.
24. How does DMA interact with the MMU (Memory Management Unit).
25. What are descriptor-based DMA engines.
26. Explain memory-to-peripheral transfer using DMA.
27. What is DMA chaining.
28. What is the role of linked list descriptors in DMA.
29. How do you ensure synchronization between CPU and DMA.
30. What is the role of DMA FIFO (First In First Out) buffers.
31. How do you debug a failed DMA transaction.
32. What is a typical size limitation for DMA transfers.
33. How do you handle unaligned memory in DMA.
34. What is the difference between blocking and non-blocking DMA APIs.
35. Explain the role of bounce buffers in DMA.

---

### Advanced/Expert Level (36–50)

Deep architectural, performance tuning, and driver-level questions.

36. How does DMA operate in a system with multiple bus masters.
37. How do you implement a zero-copy DMA system.
38. How is DMA managed in a multicore system.
39. Explain how DMA interacts with cacheable and non-cacheable memory regions.
40. What are the security risks of DMA (e.g., DMA attacks).
41. How do you protect a DMA controller from accessing illegal memory.
42. How does Linux handle DMA using the DMA mapping API.
43. What is dma_map_single() vs. dma_map_sg() in Linux.
44. How do IOMMU and DMA relate.
45. How would you implement a custom DMA driver in Linux or Baremetal.
46. What are the differences between coherent and streaming DMA mappings.
47. How do DMA engines in PCIe or AXI interconnects work.
48. How would you handle DMA transfer timeout and recovery.
49. Explain the role of dma_alloc_coherent() and its implications.
50. How do you benchmark DMA throughput and latency effectively?

---

