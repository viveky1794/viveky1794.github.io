---
layout: default
title: Misc & Kernel Boot
parent: Linux Internals
grand_parent: Articles
nav_order: 9
description: "Grab-bag of Linux internals notes: kernel boot sequence, ELF loading, module dependency resolution, deadlocks, and fork/exec/clone."
redirect_from:
  - /linux-internals/misc-and-boot/
  - /linux-internals/misc-and-boot.html
---

# Misc & Kernel Boot

Notes that didn't fit neatly into the other sections: how the kernel boots,
how ELF binaries get loaded, deferred driver probing, and the practical
differences between `fork()`, `exec()`, and `clone()`.

## Kernel Boot, ELF Loading & Process Creation

### Deadlock

**How deadlocks are recovered in Linux?**
**How to detect and find out if a program is in deadlock?**
**Are there some tools that can be used to do that on Linux/Unix systems?**

**Deadlocks are not recovered they are avoided**. Once you have a deadlock (caused by wrong programming) the only<br> solution is to kill the running program and fix it.

If you suspect a deadlock, do a `ps aux | grep <exe name>`, if in output, the `PROCESS STATE CODE` is `D`<br> (Uninterruptible sleep) means it is a deadlock.

Because as @daijo explained, say you have two threads `T1` & `T2` and two critical sections each protected by `semaphores S1 & S2` then if `T1` acquires
`S1` and `T2` acquires `S2` and after that they try to acquire the other lock before relinquishing the one already held by them, this will lead to a deadlock
 and on doing a `ps aux | grep <exe name>`, the `process state code` will be `D` (ie Uninterruptible sleep).

**Tools:**
Valgrind, Lockdep (linux kernel utility)

### Linux kernel Boot

**1. Bootloader prints "Starting kernel..."**

* This is the last message from the bootloader (e.g., U-Boot).
* It sets up:
  * Kernel image in memory
  * Device Tree Blob (DTB)
  * Initrd/initramfs (optional)
* It sets CPU to correct mode (e.g., EL2/EL1 on ARM64).
* Then it jumps to the  **kernel's entry point** .

**2. Kernel Entry Point in Assembly**

* This is architecture-specific (`arch/arm64/kernel/head.S`).
* It sets up:
  * Stack pointer
  * Basic page tables
  * Exception vector `(Load VBAR_EL1 with virtual vector table address)`
* Turns on **MMU** and  **caches** .
* Then jumps to `start_kernel()` in C.

**3. `start_kernel()` function begins (in C)**

* Located in `init/main.c`.
* It is the first high-level C function in Linux.

**4. `start_kernel()` key steps (highlights):**

> You can memorize these 7 stages:

**(a)** `setup_arch()`

– Parses device tree, memory layout, and CPU info.

**(b)** `mm_init()`

– Sets up memory management and page allocators.

**(c)** `console_init()`

– Sets up early console and `printk`.

**(d)** `printk(linux_banner)`

– **This prints the Linux version banner.**

**(e)** `trap_init()`, `sched_init()`

– Initializes traps, scheduler, timekeeping.

**(f)** `rest_init()`

– Creates the first kernel thread (`init`).

**(g)** `kernel_init()`

– Mounts root filesystem and launches user-space `init`.

#### Final Answer You Can Say

> After `Starting kernel...`, the bootloader jumps to the Linux kernel's entry point in assembly (head.S).<br> It sets up the stack, enables MMU, and switches to virtual addressing. Then it jumps to `start_kernel()` in C,<br> which handles memory setup, interrupt controller, and early console.<br> During this, the `printk` function prints the Linux version banner. That's the first message seen from the kernel itself.

#### Post-Banner Boot Process

“After the Linux banner is printed, the kernel continues initializing core subsystems.

It sets up interrupts, scheduler, memory zones, and the root filesystem.

Then, it creates the first user-space process by calling `kernel_init()`,

which eventually executes `/sbin/init`, or the init process from initramfs.

From there, user-space starts, and system services come up.”

### How Programs and Libraries are Loaded into Memory

How Programs and Libraries are Loaded into Memory (Linux, ELF format).

When you run a program: `$./my_program` the **kernel loader** (`execve()` syscall) does the job of loading the program and required libraries into memory.

🧠 Step-by-Step: Program Loading:

**1. Disk to Memory (via ELF format)**
Linux binaries (like ./my_program) are ELF (Executable and Linkable Format) files.

> The ELF file contains:
> Code (.text)
> Initialized data (.data)
> Uninitialized data (.bss)
> Symbol tables, headers
> Dynamic linking info

**2. The OS loader (kernel space) does:**
Step	Description
1️⃣	Parses ELF headers
2️⃣	Allocates memory regions for text, data, bss
3️⃣	Loads .text segment as read-only, executable
4️⃣	Loads .data segment as read-write
5️⃣	Allocates .bss and initializes to zero
6️⃣	Sets up stack (with argc, argv, envp)
7️⃣	Maps heap starting after data
8️⃣	Resolves dynamic libraries (e.g., libc.so)
9️⃣	Jumps to entry point (from ELF header)

✅ All of this is triggered by `execve()` under the hood.

**Dynamic Libraries Loading (.so files)**
![alt text](/assets/images/notes/Linux_misc/image-1.png)

#### Section-2

✅ Microcontrollers (Bare-metal systems)
In bare-metal systems (no OS) — like STM32, AVR, etc.:

The Flash memory contains the entire program.

Upon reset, startup code (bootloader or crt0.s) does this:
![1749214449665](/assets/images/notes/Linux_IPC_socket/1749214449665.png)

🔹 Why is .text executed from Flash?
Because:

Flash is memory-mapped into the address space

MCU executes code directly from it (e.g., at address 0x08000000)

No need to copy .text to SRAM (saves RAM space)

📌 Advantage: Saves SRAM, smaller footprint.

---

✅ Linux (with MMU and virtual memory)
In contrast, on Linux:

ELF executables are loaded from storage (disk, SSD) into virtual memory by the kernel

The .text, .data, .bss sections are mapped into memory (using mmap)

🔹 What happens to .text section?
It is mapped into memory (typically from the file-backed ELF via mmap)

Pages are marked read-only and executable

So it appears as if .text "came to memory", but it’s actually lazy-loaded (demand paging)

![1749214511297](/assets/images/notes/Linux_IPC_socket/1749214511297.png)

📌 Linux needs .text in RAM because:

Code must be paged in and managed by the MMU

Linux supports dynamic linking, relocation, etc.

| Feature                       | Microcontroller (Bare-metal) | Linux (MMU-based system)        |
| ----------------------------- | ---------------------------- | ------------------------------- |
| `.text`                     | Executed from Flash          | Loaded into memory (via mmap)   |
| `.data`                     | Copied from Flash to SRAM    | Mapped into memory (read/write) |
| `.bss`                      | Zero-initialized in SRAM     | Zeroed memory (anonymous mmap)  |
| Code execution                | Direct from Flash            | From virtual memory (RAM)       |
| MMU / paging                  | ❌ No                        | ✅ Yes                          |
| Bootloader initializes memory | ✅ Yes (startup code)        | ❌ Kernel/loader does it        |

🔧 Extra: Embedded Linux on SoCs
In embedded Linux on SoCs (e.g., Cortex-A):

.text is loaded from Flash (e.g., eMMC) into RAM during boot

U-Boot or TF-A loads kernel ELF or uImage into DRAM

No code is executed directly from Flash — everything runs from DRAM

**If all .text section of all the programs copied into RAM than RAM will not have sapce after some time.**

❓ Do all .text sections of all programs stay in RAM and consume space permanently?
➡️ No, and here's why RAM doesn't get full:

📌 Key Points to Say in Interview
Linux doesn't copy full .text sections into RAM upfront — it uses demand paging to load code only when needed.

Code sections (.text) of shared libraries (like libc) are shared among processes — one copy in RAM.

The .text section is memory-mapped directly from disk (ELF), not duplicated.

Clean, unused .text pages can be evicted anytime (no swap needed) and reloaded from disk when required.

This makes RAM use efficient and scalable, even with many processes.

> The .text section is memory-mapped directly from disk (ELF), not duplicated.
> If above line is correct than what happens if the backing ELF file is deleted while the program is still running.
>
> ## 📌 But if you delete the ELF file?
>
> Deleting a file in Linux  **doesn't remove the data immediately** . Instead:
>
> * The  **directory entry is removed** , but
> * The **inode and data blocks remain** if any process still has it **open** (including through `mmap`).
>
> That means:
>
> ✅ **The mapped `.text` section remains valid** in memory
>
> ✅ **The program continues to run normally**
>
> ✅ The program still runs
> ✅ No crash
> ✅ No .text fetch failure
>
> ❌ When could problems occur?
> Only in advanced cases:
>
> | Scenario                                    | Result                                                                               |
> | ------------------------------------------- | ------------------------------------------------------------------------------------ |
> | Page not yet loaded, and deleted ELF        | Kernel might try to fetch missing `.text` page → **fail or SIGSEGV**        |
> | Stripped or truncated file during execution | Could cause crashes (depends on which pages are needed and whether they were loaded) |
> | File overwritten with something else        | Corruption risk increases                                                            |
> | Reboot system                               | Then file is gone, program can’t be started again                                   |

**🧠 Summary (Interview-Ready)**
"Deleting the ELF file of a running program in Linux doesn't crash it immediately because memory-mapped .text pages remain valid as long as they're in memory or the inode is open — but if the program accesses pages that weren’t loaded yet, it can crash."

### Memory Segmentation

Segmentation is a memory management technique used by operating systems to divide a program's memory into logical segments such as code, data, stack, etc. Each segment represents a specific type of content or usage and is addressed separately.

🔍 **1. Why Segmentation?**
Earlier memory models used flat addressing (a single continuous memory space). Segmentation introduced the idea of dividing memory logically to:
Support modularity in programming (code/data separation)
Provide protection (e.g., read-only code segment)
Simplify memory sharing between processes
Handle growing and shrinking segments independently (e.g., stack grows down, heap grows up)
Each segment is handled and protected separately.

#### Section-8

🧠 Visual Example
int *arr = malloc(4 * sizeof(int));  // allocates 16 bytes
┌──────────┬────────────────────────────┐
│ Metadata │ Usable Block: 16 bytes     │ ← You only see this
└──────────┴────────────────────────────┘

⚠️ Common Pitfalls and Mistakes:

| Mistake                                       | Explanation                                                  |
| --------------------------------------------- | ------------------------------------------------------------ |
| **Forgetting `free()`**               | Causes memory leaks.                                         |
| **Double `free()`**                   | Causes undefined behavior — may crash or corrupt heap.      |
| **Using after `free()`**              | Called**dangling pointer** — very dangerous.          |
| **Not checking `malloc()` result**    | If memory is exhausted, it returns `NULL`.                 |
| **Assuming memory is zeroed**           | `malloc()` gives garbage data unless you use `calloc()`. |
| **Buffer overflows**                    | Writing past the allocated size corrupts the heap.           |
| **Mixing `malloc()` with `delete`** | Mixing C/C++ memory models is undefined behavior.            |

🧪 Diagnosing malloc Issues:

| Tool                 | Purpose                                          |
| -------------------- | ------------------------------------------------ |
| `valgrind`         | Detect memory leaks, invalid accesses            |
| `asan` (GCC/Clang) | Catch overflows, use-after-free                  |
| `gdb`              | Inspect heap, set breakpoints on `malloc/free` |
| `mtrace()`         | Trace malloc/free behavior                       |

| Function      | Use Case                                               |
| ------------- | ------------------------------------------------------ |
| `malloc()`  | Fast, uninitialized memory                             |
| `calloc()`  | Zeroed memory                                          |
| `realloc()` | Resize while preserving contents                       |
| `free()`    | Always match every `malloc`/`calloc` with `free` |

### Difference between fork, exec and clone

🔁 fork(), exec(), clone() – At a Glance

| System Call | What It Does                            | Memory Inheritance               | Common Use                                     |
| ----------- | --------------------------------------- | -------------------------------- | ---------------------------------------------- |
| `fork()`  | Duplicates current process              | Copies all memory (COW)          | Traditional UNIX process creation              |
| `exec()`  | Replaces process image with new program | Drops current memory             | Load new program into process                  |
| `clone()` | Customizable process/thread creation    | Share or copy memory selectively | Used in `pthread_create()`, namespaces, etc. |

🧠 fork() – Copy-on-Write Process Duplication
What it does:
Creates a child process that is a copy of the parent, including:

  Memory (stack, heap, data, bss, mmap, etc.)

  File descriptors

  Signals

  Environment, etc.

But Linux uses Copy-on-Write (COW):

  Parent and child share physical memory pages

  When one writes → the page is copied

Memory View:
Parent             Child
 ──────            ──────
 Text   ───▶ shared
 Heap   ───▶ shared (COW)
 Stack  ───▶ shared (COW)

🚀 exec() – Replace the Process Image
What it does:
  Replaces the calling process image with a new program.

  Memory layout is completely discarded and replaced.

> execl("/bin/ls", "ls", "-l", NULL);

After exec():

> New program runs in the same PID.
> Memory: text, stack, heap, env — all new.

⚙️ clone() – Flexible Creation (used in Threads)
`clone()` is lower-level and more flexible than fork().

You can control what is shared between parent and child:
Common flags:

| Flag                               | Description                                 |
| ---------------------------------- | ------------------------------------------- |
| `CLONE_VM`                       | Share memory space (like threads)           |
| `CLONE_FS`                       | Share filesystem info                       |
| `CLONE_FILES`                    | Share file descriptors                      |
| `CLONE_THREAD`                   | Same thread group (like `pthread_create`) |
| `CLONE_NEWUTS`, `CLONE_NEWPID` | Create namespaces (containers!)             |

🔐 Bonus: Memory layout after fork()
After fork():

  Parent and child have identical virtual memory maps.
  Physical pages are shared with COW:
    Read → shared
    Write → page is duplicated
  Changes in one do not affect the other (unless mmap(MAP_SHARED))

 ----------------

### How does system call works ?

![1751208033953](/assets/images/notes/Linux_misc/1751208033953.png)
![1751208045356](/assets/images/notes/Linux_misc/1751208045356.png)
![1751208052346](/assets/images/notes/Linux_misc/1751208052346.png)
![1751208068256](/assets/images/notes/Linux_misc/1751208068256.png)

## Kernel Module Dependency Resolution (Deferred Probing)

### Linux kernel Module dependecy

Ever wondered how Linux gracefully handles device dependencies during boot when driver probe order isn't predictable?

The Problem: Device discovery (e.g., via Device Tree) doesn't guarantee dependency order. A driver might probe before its required resources—clocks, regulators, GPIO controllers—are ready.

The Solution: Deferred Probing - When a driver needs an unavailable resource, it returns -EPROBE_DEFER instead of failing.

The kernel then:

1. Moves the driver to a deferred list
2. Retries after each successful probe (when new resources become available)
3. Eventually, it times out to prevent infinite loops

This mechanism transforms potential boot failures into graceful dependency resolution, allowing complex hardware to initialize reliably regardless of discovery order.

I once debugged a driver that seemed to have perfect DTS matching but whose probe() never seemed to execute. The culprit? Silent -EPROBE_DEFER returns due to a broken clock driver dependency. The driver kept deferring indefinitely without logs, making it appear as if probe() wasn't being called at all!

![1749037601812](/assets/images/notes/Linux_module/1749037601812.png)


## Interview Question Bank: Sockets & Memory Management

A running list of interview/self-study questions on socket programming and
Linux memory management, collected while prepping for embedded/systems
interviews. Left as an unanswered checklist deliberately — treat it as
practice material, not a reference. Items already covered elsewhere on this
site are linked.

**Socket programming:**

- What is a socket, and what is it used for?
- How are IP addresses resolved? What's the difference between IPv4 and
  IPv6, and how does that affect the socket calls?
- How do you determine what port to connect to? What are common ports, and
  which ones would you avoid?
- TCP socket vs. UDP socket — what's the difference, and when would you use
  each?
- What sequence of calls establishes a TCP and/or UDP connection, client
  side and server side? What information does each call need?
- How do you test whether a socket is ready to be read or written?
- If a socket closes unexpectedly, how would you detect and handle it?
- What happens to the connection/thread/process if a socket isn't closed
  properly?
- How long does an idle socket connection stay open, and how would you
  change that?
- How do you set a socket to non-blocking mode, and when would you want to?
- What are the considerations when writing a multi-process and/or
  multi-threaded socket application? Is there more than one way to handle
  it?

**Memory management — knowledge questions:**

- Difference between logical, physical, and virtual memory in Linux.
- When do you use `ioremap` vs. `mmap`?
- Given a known physical address (a register or device address), how do
  you access it from user space or a kernel driver?
- What is high/low memory in Linux?
- Given a connected PCIe device, how do you access its memory from
  application code?
- What is a memory map? Describe one from a system you know well.
- What problem arises when DMA operates on cacheable memory, and how do
  you solve it? (See [DMA](/articles/linux/dma).)
- Page tables and how they're handled.
- L1/L2 cache in Linux. (See [Caching](/articles/linux/caching).)
- Describe the memory hierarchy (asked in nearly every ops interview).
- Basic SMP architecture — be able to diagram it.
- How is memory addressed?
- [How are programs and libraries loaded into memory?](#kernel-boot-elf-loading-process-creation)
- On a microcontroller, `.text` executes directly from Flash while `.data`
  and `.bss` load into SRAM — but on Linux, `.text` is also loaded into RAM
  as part of the virtual memory mapping. Why?
- [What is segmentation?](#memory-segmentation)
- How does the kernel manage its own memory? Kernel memory vs. user memory?
  What is shared memory?
- What is paging — what hardware and software mechanisms are involved?
- What's the difference between physical and virtual memory?
- What is swapping, and how does it relate to paging?
- What is thrashing?
- What are page faults?
- The `malloc` family: what do these functions do, how do they work, and
  what are the common pitfalls?
- The `fork`/`exec`/`clone` family and how each relates to process memory.
  (See [fork, exec & clone](#difference-between-fork-exec-and-clone).)
- POSIX shared memory: `mmap`, `shm_open`, `fstat`, and similar.
- What is DMA, and how does it work? (See [DMA](/articles/linux/dma).)
- What is a cache vs. a buffer — compare and contrast, with examples of
  where you'd encounter each. (See [Caching](/articles/linux/caching).)

**Memory management — practical questions:**

- What tools show overall memory utilization and memory-subsystem
  performance?
- How do you find an individual process's memory utilization (RSS, VSZ,
  paging, etc.)?
- Where are memory-related errors typically logged?
- What debugging tools help diagnose memory issues, and when would you
  reach for each (`strace`, `gdb`, `valgrind`, core dumps, etc.)?
- What process-memory information is available in `/proc`?
- How can memory subsystem parameters be tuned? What are common changes?
- Describe the impact of spatial and temporal locality on application
  performance.

**Memory management — special topics** (niche, but occasionally asked):

- `memtest` and similar hardware diagnostics — when should you suspect a
  hardware problem?
- What constitutes effective alerting on memory utilization?
- What is the OOM killer, and why does it matter?
- How does containerization relate to memory management?
- NUMA vs. SMP — how do they differ, and where does each show up?
- What is copy-on-write (COW), and why did it matter for Linux security
  around 2016?
- DMA vs. RDMA — where would you expect to find each?
- Compare LRU, LFU, MFU, FIFO, and similar page-replacement algorithms.
- How do Type-1 hypervisors affect a guest's memory subsystem, and where
  does that become visible? (See [QNX](/articles/qnx).)

## What Is an OS?

### What is OS?

OS is a program which manages computer resources while serving programs.
Suppose there is not an OS then every program Developer need to write its resource management code also.
In such case every Application became bulky due to duplicate code.
Which is clearly violation of **DRY**(Don't repeat yourself) laws.
OS provides **isolation & Protection**.

### Types of OS

### Difference between Multi-Processing, Multi-Tasking and Multi-Threading ??

**Process** : Program under execution

**Thread** : Light Weight Process

    It is basically a sub process of your program(main process) which can run independently.
				Threads Uses Process resources. Because they are part of a process. So the concept of OS isolation does 				not apply here.

| Multi-Tasking                                          | Multi-Threading                                                                                                    |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| It is apply only when More then one process available. | It can apply between multiple Threads(sub-process).                                                                |
| Isolation & Memory Protection ensures by OS.           | No isolation & Memory protection. It is part of a main process. So they use process resources like memory and CPU. |
| Process Scheduling                                     | Threads scheduling                                                                                                 |
|                                                        |                                                                                                                    |

