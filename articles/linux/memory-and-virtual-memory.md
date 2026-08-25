---
layout: default
title: Memory & Virtual Memory
parent: Linux Internals
grand_parent: Articles
nav_order: 2
description: "Virtual memory end to end: why it exists, page tables, address translation, the TLB, page faults, swapping/thrashing, and memory protection."
redirect_from:
  - /linux-internals/memory-and-virtual-memory/
  - /linux-internals/memory-and-virtual-memory.html
---

# Memory & Virtual Memory

## Why Virtual Memory Exists

Virtual memory is an abstraction, provided jointly by the OS and the
hardware, that solves three concrete problems:

1. **Not enough memory** — a process's working set can exceed installed
   RAM. Virtual memory lets the system run more (or larger) processes than
   physical RAM would otherwise allow, by keeping only the actively-used
   parts of a process resident and the rest on disk.
2. **Memory fragmentation** — without indirection, a process needs a single
   contiguous block of physical memory. Virtual memory lets a process's
   *logical* address space stay contiguous while the *physical* pages
   backing it are scattered anywhere in RAM.
3. **Security** — giving every process its own private address space
   prevents one process from directly reading or corrupting another's
   memory.

| | Physical Memory | Virtual Memory |
|---|---|---|
| **Definition** | Actual hardware RAM | Abstraction of memory provided by OS + hardware |
| **Location** | RAM (primary memory) | RAM and/or disk (via swap/paging) |
| **Size limit** | Limited by installed RAM (e.g. 8 GB) | Limited by CPU architecture (e.g. 48-bit VA → 256 TB) |
| **Managed by** | Hardware (memory controller) | OS + MMU |
| **Accessed by** | OS kernel directly | User processes and kernel, via virtual addresses |
| **Address type** | Physical addresses (PA) | Virtual addresses (VA) |
| **Isolation** | None across processes | Each process gets its own address space |
| **Swappable?** | No | Yes (RAM ↔ disk) |

![1749364366483](/assets/images/notes/Linux_IPC_socket/1749364366483.png)
![1749364615901](/assets/images/notes/Linux_IPC_socket/1749364615901.png)
![1749364799372](/assets/images/notes/Linux_IPC_socket/1749364799372.png)
![1749396328022](/assets/images/notes/Linux_IPC_socket/1749396328022.png)
![alt text](/assets/images/notes/image-2.png)

### The Three Problems, Concretely

**Not enough memory:** early CPUs with 32-bit registers could address at
most 4 GB, and installed RAM was often far smaller still. A machine with
2 GB of RAM (a 31-bit space) has no problem with the first few bytes a
program touches — but a program trying to use more than 2 GB simply
crashes, since there's nowhere for that memory to physically exist. This is
why programs could crash outright from running out of memory in early
computing.

![1751176043392](/assets/images/notes/Linux_virtual_memory_part_1/1751176043392.png)
![1751176189287](/assets/images/notes/Linux_virtual_memory_part_1/1751176189287.png)
![1751176199219](/assets/images/notes/Linux_virtual_memory_part_1/1751176199219.png)
![1751176229572](/assets/images/notes/Linux_virtual_memory_part_1/1751176229572.png)
![1751176252686](/assets/images/notes/Linux_virtual_memory_part_1/1751176252686.png)

**Memory fragmentation:** with 4 GB of RAM and three programs — a 1 GB
video player, a 2 GB game, and a 2 GB photo editor — running the player and
the game fits (3 GB used, 1 GB free). Closing the player frees 1 GB more,
for 2 GB free total — but split into two separate 1 GB chunks. The photo
editor needs 2 GB *contiguous*, and can't run, even though there's
technically enough free memory in total. Splitting a program's own memory
across those chunks is possible in principle, but writing programs that
account for that manually — no straight-line indexing into an array that
might be split across two disjoint regions — is exactly the kind of problem
an indirection layer (virtual memory) exists to hide.

![1751176344256](/assets/images/notes/Linux_virtual_memory_part_1/1751176344256.png)
![1751176371483](/assets/images/notes/Linux_virtual_memory_part_1/1751176371483.png)

**Data corruption / security:** if every program can address the same
physical memory directly, two unrelated programs can collide. Imagine a
video game storing the player's health at address 64, while a music player
happens to write the remaining track duration to that same address 64 —
when the song ends, the music player zeroes that address, and the game
reads it as zero health. Two completely unrelated programs corrupting each
other's state, simply because they share one flat address space.

![1751176423989](/assets/images/notes/Linux_virtual_memory_part_1/1751176423989.png)
![1751176538308](/assets/images/notes/Linux_virtual_memory_part_1/1751176538308.png)

All three problems trace back to the same root cause: every program sharing
one physical address space. Give each program its own private address
space instead, with a layer that maps it to physical memory, and all three
become solvable.

### Indirection: The Core Idea

The address space assigned to each program is its **virtual memory** — one
per program, non-overlapping with any other program's. Making that useful
requires mapping each program's virtual addresses to real **physical**
addresses (RAM addresses, plus potentially the addresses of other
memory-mapped devices — not just RAM, which is why the general term is
"physical," not "RAM").

![1751176632286](/assets/images/notes/Linux_virtual_memory_part_1/1751176632286.png)
![1751176643620](/assets/images/notes/Linux_virtual_memory_part_1/1751176643620.png)

At boot, installed RAM becomes available to the OS starting from some base
physical address; the OS reserves part of it for itself and hands the rest
to programs.

![1751176697576](/assets/images/notes/Linux_virtual_memory_part_1/1751176697576.png)
![1751176717880](/assets/images/notes/Linux_virtual_memory_part_1/1751176717880.png)

Without virtual memory, a program's address space maps directly onto RAM.
With it, a **map** sits in the middle: a program addressing virtual
address 0 might actually be reading physical address 5 — any virtual
address can map to any physical address, entirely opaque to the program.

![1751176785711](/assets/images/notes/Linux_virtual_memory_part_1/1751176785711.png)
![1751176806834](/assets/images/notes/Linux_virtual_memory_part_1/1751176806834.png)

That same indirection is what makes **swap memory** possible: if a program
needs more data than fits in RAM, the OS evicts something else (the oldest
data, say), writes it to disk, loads what's actually needed, and updates
the map — using disk as overflow capacity for RAM. Reading from disk
instead of RAM is exactly what a **page fault** is (more below) — and it's
slow: even an SSD's latency to the first byte can run roughly a thousand
times worse than RAM's. This is the direct reason more RAM measurably
speeds up a machine that's swapping heavily — less swapping, fewer page
faults.

![1751176921966](/assets/images/notes/Linux_virtual_memory_part_1/1751176921966.png)
![1751176944259](/assets/images/notes/Linux_virtual_memory_part_1/1751176944259.png)
![1751176960792](/assets/images/notes/Linux_virtual_memory_part_1/1751176960792.png)
![1751177031137](/assets/images/notes/Linux_virtual_memory_part_1/1751177031137.png)

The same indirection fixes fragmentation directly: the earlier example (a
game running, then trying to also run a photo editor in two disjoint 1 GB
free chunks) can now map the editor's *contiguous* virtual address range
onto those two disjoint physical chunks — the program still sees one
continuous space and never knows the difference.

![1751177198134](/assets/images/notes/Linux_virtual_memory_part_1/1751177198134.png)

And it fixes the security problem the same way: the video game and the
music player each get their own map. Both may write to virtual address 64,
but the game's map sends that to physical address 10 while the music
player's sends it to physical address 4 — no collision, despite identical
virtual addresses. Total isolation would also block legitimate data
sharing, so the same mechanism supports the opposite case deliberately: two
programs *can* have parts of their address spaces mapped to the *same*
physical memory — how a shared library (libc, a UI toolkit) or a shared
memory segment for fast IPC actually works.

![1751178498235](/assets/images/notes/Linux_virtual_memory_part_1/1751178498235.png)
![1751178516236](/assets/images/notes/Linux_virtual_memory_part_1/1751178516236.png)

## Page Tables

The OS maps virtual pages to physical frames using a **page table**: one
entry per page, holding that page's physical address in RAM plus metadata
(protection bits, present/dirty status, etc).

The number of entries adds up fast. At 4 KB pages, 16 GB of RAM holds 4
million pages — and since each process gets its *own* page table, a 4-byte
page table entry means ~16 MB of page table per process. With 50 processes
running, that's ~800 MB spent purely on page tables — not acceptable on its
own, which is part of why real page tables are multi-level (a tree of
smaller tables, not one giant flat array) rather than the simplified flat
model used for intuition here.

![alt text](/assets/images/notes/image-4.png)
![1749313255324](/assets/images/notes/Linux_IPC_socket/1749313255324.png)
![1749313539369](/assets/images/notes/Linux_IPC_socket/1749313539369.png)
![1749313508331](/assets/images/notes/Linux_IPC_socket/1749313508331.png)
![1749313470308](/assets/images/notes/Linux_IPC_socket/1749313470308.png)

### Why Pages, Not Individual Words

The naive version of a page table maps every individually-addressable unit
— every CPU word (4 bytes on a 32-bit machine) — to a physical address on
its own. With a 32-bit address space, that's 2³⁰ words needing an entry
each; at 4 bytes per entry, that's a 4 GB page table — *per program*. Not
remotely workable.

![1751178606438](/assets/images/notes/Linux_virtual_memory_part_1/1751178606438.png)
![1751178660901](/assets/images/notes/Linux_virtual_memory_part_1/1751178660901.png)
![1751179029309](/assets/images/notes/Linux_virtual_memory_part_1/1751179029309.png)

The fix is exactly the page/frame chunking already introduced above:
instead of one entry per word, one entry covers an entire 4 KB chunk (1,024
words) — e.g. virtual addresses 0–4,095 might map as a block onto physical
addresses 16,384–20,479. That drops the page table to roughly a million
entries at 4 bytes each — a few megabytes, per program, instead of 4 GB.
The trade-off: evicting or loading memory now moves a whole 4 KB page at a
time instead of a single word — which works well in practice, since nearby
memory tends to get accessed together anyway (the same locality of
reference caching relies on).

![1751179277432](/assets/images/notes/Linux_virtual_memory_part_1/1751179277432.png)

Mapping a page means mapping a *range*: virtual page 1 (addresses
4,096–8,191) might map to physical page 2 (addresses 8,192–12,287). Any
address inside that virtual page keeps the *same offset* from the page
start in the physical page — virtual address 4,200 is 104 bytes into
virtual page 1, so it lands at physical address 8,192 + 104 = **8,296**.

![1751179492583](/assets/images/notes/Linux_virtual_memory_part_1/1751179492583.png)

### Multi-Level Page Tables

Even at a few megabytes per program, page tables add up: 50 programs at
roughly 4 MB of page table each is ~200 MB just for translation metadata —
and a real desktop running hundreds of background programs (595, in one
concrete case) could need 2+ GB of RAM purely for page tables, most of it
for programs barely using any memory themselves.

![1751183111653](/assets/images/notes/Linux_virtual_memory_part_1/1751183111653.png)

The obvious fix — swap unused page tables to disk like any other memory —
runs into a chicken-and-egg problem: the CPU needs the page table *to find*
anything in memory, including the page table itself if it's not resident.
The real solution is a **second level of indirection**: split the ~1
million page table entries (for 4 KB pages) into 4 KB chunks of their own
(1,024 "second-level" pages of translations), and add a small
**first-level** table that maps a virtual address to *which* second-level
chunk holds its real translation. The first-level table always stays
resident in RAM; second-level chunks — like any other page — can be
swapped out when not needed. Real architectures aren't limited to two
levels — Linux uses **five-level page tables** specifically to address
today's 64 TB-class server RAM configurations, at the cost of more memory
accesses per translation as levels increase.

![1751183268354](/assets/images/notes/Linux_virtual_memory_part_1/1751183268354.png)
![1751183389584](/assets/images/notes/Linux_virtual_memory_part_1/1751183389584.png)
![1751183441643](/assets/images/notes/Linux_virtual_memory_part_1/1751183441643.png)

**Two-level translation, worked through:** with a 32-bit virtual address, a
30-bit physical address (1 GB RAM), and 4 KB pages, the low 12 bits are
still the untranslated offset, same as always. The remaining 20-bit virtual
page number splits into two 10-bit halves: the first 10 bits index into the
(always-resident) first-level table, which points at *which* second-level
table holds the real entry; the second 10 bits then index into that
second-level table to get the actual physical page number. Only the
first-level table is guaranteed to be in RAM — individual second-level
tables can be swapped out, which is exactly the win for a program that only
touches a small fraction of its full virtual address space: the
second-level tables for the parts it never touches simply never need to be
resident.

![1751183612068](/assets/images/notes/Linux_virtual_memory_part_1/1751183612068.png)

## Address Translation: Page Number & Offset

Looking up the full page table in RAM on *every* memory access would double
(or worse) every access — unacceptable given the CPU already averages more
than one memory access per instruction. This is what makes a hardware
**MMU (Memory Management Unit)** necessary: it sits on the translation path
between the CPU and memory, translates every virtual address to a physical
one, and enforces that a process can't reach memory it doesn't own.

![1749313809394](/assets/images/notes/Linux_IPC_socket/1749313809394.png)
![alt text](/assets/images/notes/image-3.png)
![1749314138274](/assets/images/notes/Linux_IPC_socket/1749314138274.png)
![1749314415297](/assets/images/notes/Linux_IPC_socket/1749314415297.png)
![1749314473340](/assets/images/notes/Linux_IPC_socket/1749314473340.png)
![1749314498915](/assets/images/notes/Linux_IPC_socket/1749314498915.png)

Every virtual address splits into two fields:

- **Page number** — the sequential page index in virtual memory (translated).
- **Page offset** — the byte position within that page (never translated —
  it's copied straight into the physical address).

![1749314898684](/assets/images/notes/Linux_IPC_socket/1749314898684.png)
![1749315026718](/assets/images/notes/Linux_IPC_socket/1749315026718.png)

**Worked example** (32-bit virtual address space = 4 GB, 256 MB physical
RAM, 4 KB pages):

- A 4 KB page needs 12 bits for the offset (2¹² = 4096), so those bottom 12
  bits of both the virtual and physical address are identical and
  untranslated.
- The remaining upper **20 bits** of the 32-bit virtual address form the
  **virtual page number (VPN)**.
- Since physical RAM is only 256 MB (28-bit), the physical address needs
  only a **16-bit physical page number (PPN)** — the page table's real job
  is translating a 20-bit VPN down to a 16-bit PPN.
- Concretely: virtual address `0x00003204` → the page table looks up its
  VPN → translates to physical address `0x00000624` (assuming that page is
  resident).

Each page table entry (PTE) holds either the physical page number (page is
in RAM) or a marker that the page lives on disk. Since there are more
virtual pages than physical frames, some virtual pages are necessarily on
disk rather than RAM at any given moment — the mismatch the page table
exists to manage.

Another worked bit-level example, this time with a 32-bit virtual address,
30-bit physical (1 GB RAM), and 4 KB pages: translating virtual address
`12345678` keeps the last 12 bits (`678`) unchanged as the offset, looks up
virtual page number `12345` in the page table to get back physical page
number `0432`, then concatenates `0432` + `678` for the full physical
address.

![1751180066944](/assets/images/notes/Linux_virtual_memory_part_1/1751180066944.png)
![1751180073245](/assets/images/notes/Linux_virtual_memory_part_1/1751180073245.png)
![1751180086897](/assets/images/notes/Linux_virtual_memory_part_1/1751180086897.png)
![1751180094248](/assets/images/notes/Linux_virtual_memory_part_1/1751180094248.png)
![1751180114311](/assets/images/notes/Linux_virtual_memory_part_1/1751180114311.png)
![1751180350662](/assets/images/notes/Linux_virtual_memory_part_1/1751180350662.png)
![1751180387352](/assets/images/notes/Linux_virtual_memory_part_1/1751180387352.png)

![1751774777472](/assets/images/notes/Linux_virtual_memory_part_2/1751774777472.png)
![1751774831799](/assets/images/notes/Linux_virtual_memory_part_2/1751774831799.png)
![1751775003331](/assets/images/notes/Linux_virtual_memory_part_2/1751775003331.png)
![1751775189973](/assets/images/notes/Linux_virtual_memory_part_2/1751775189973.png)
![1751776721465](/assets/images/notes/Linux_virtual_memory_part_3/1751776721465.png)
![1751776748375](/assets/images/notes/Linux_virtual_memory_part_3/1751776748375.png)
![1751776810616](/assets/images/notes/Linux_virtual_memory_part_3/1751776810616.png)
![1751776876854](/assets/images/notes/Linux_virtual_memory_part_3/1751776876854.png)
![1751776994048](/assets/images/notes/Linux_virtual_memory_part_3/1751776994048.png)

**Page size is a trade-off.** Moving from 4 KB to 64 KB pages means the
offset grows to 16 bits (2¹⁶ = 64 KB) and the VPN shrinks accordingly —
fewer page table entries are needed to cover the same address space, so the
table itself shrinks. The cost is coarser granularity: more data has to be
loaded or evicted together per page, increasing internal fragmentation and
the amount of unrelated data dragged along on every fault.

## The TLB (Translation Lookaside Buffer)

A page table walk in RAM on every access is exactly the overhead the MMU
alone can't avoid — so hardware adds a small, extremely fast cache of
recent virtual→physical translations sitting directly on the CPU's memory
path: the **TLB**.

**TLB Entries** are similar to page table entries but add an **ASID**
(Address Space ID) that tags which process a given entry belongs to.
Without ASID, every context switch would force a full TLB flush and
repopulation for the incoming process — defeating much of the point of
having a TLB at all.

![1751181313323](/assets/images/notes/Linux_virtual_memory_part_1/1751181313323.png)
![1751181574628](/assets/images/notes/Linux_virtual_memory_part_1/1751181574628.png)
![1751181584096](/assets/images/notes/Linux_virtual_memory_part_1/1751181584096.png)
![1751181622356](/assets/images/notes/Linux_virtual_memory_part_1/1751181622356.png)

Modern architectures keep TLBs small — often only a few thousand entries —
which is exactly why they're constantly being refilled; this only works
well in practice because of locality of reference (see
[Caching](/articles/linux/caching)). The piece of hardware actually
responsible for translation and for raising page faults is the **MMU**
(Memory Management Unit) itself — the TLB is its fast-path cache, sitting
on the CPU package and programmed by the OS.

![1751183050431](/assets/images/notes/Linux_virtual_memory_part_1/1751183050431.png)
![1751183058967](/assets/images/notes/Linux_virtual_memory_part_1/1751183058967.png)

![1749314558327](/assets/images/notes/Linux_IPC_socket/1749314558327.png)
![1749315091979](/assets/images/notes/Linux_IPC_socket/1749315091979.png)
![1749315174668](/assets/images/notes/Linux_IPC_socket/1749315174668.png)

**Typical TLB characteristics:**

- Very fast — one cycle or less.
- Small — often ~64 entries.
- Frequently split into a separate instruction TLB (I-TLB) and data TLB
  (D-TLB).
- Commonly 4-way set associative; at 64 entries × 4 KB pages, that's 256 KB
  of address space covered per TLB.

**On a TLB miss:**

- **Page is in RAM** — the full page table in memory has to be walked
  (~20–1,000 cycles: slow relative to a TLB hit, but far cheaper than a page
  fault), and the TLB is updated with the new entry before retrying.
- **Page is on disk** — a full **page fault** (~80 million cycles — see
  below), since the OS has to swap the page in before anything can proceed.

![1749315405863](/assets/images/notes/Linux_IPC_socket/1749315405863.png)
![1749315454726](/assets/images/notes/Linux_IPC_socket/1749315454726.png)
![1749315545014](/assets/images/notes/Linux_IPC_socket/1749315545014.png)
![1749315629724](/assets/images/notes/Linux_IPC_socket/1749315629724.png)
![1749315732924](/assets/images/notes/Linux_IPC_socket/1749315732924.png)

**Improving TLB coverage and performance:**

1. **Larger pages** (e.g. 2 MB instead of 4 KB) — fewer TLB entries needed
   to cover the same amount of memory, at the cost of more internal
   fragmentation and coarser eviction/swap granularity.
2. **Multi-level TLBs** — a small, very fast L1 TLB (~64 entries) backed by
   a larger, slower L2 TLB (~512 entries); an L2 lookup is still far
   cheaper than a full page table walk in RAM.
3. **Hardware page table walker** — on a miss, dedicated hardware walks the
   page table directly instead of trapping into the OS, which is
   significantly faster than a software-only walk. Some designs go further
   and add a DMA-like path specifically for pulling translations from RAM
   into the TLB without looping back through the OS at all.

![1751181630871](/assets/images/notes/Linux_virtual_memory_part_1/1751181630871.png)
![1751182894971](/assets/images/notes/Linux_virtual_memory_part_1/1751182894971.png)

**Worked example:** translating virtual address `12345678` again, but now
through the TLB — the offset (`678`) is copied straight through as always;
the virtual page number `12345` is looked up in the TLB first. On an empty
TLB, that misses, so the CPU walks the real page table in RAM, finds the
mapping, and *also* writes it into the TLB before completing the
translation. The next access to any address sharing that same virtual page
number hits directly in the TLB. On a full TLB, the least-recently-used
entry is evicted to make room — and if the target page turns out to be on
disk rather than RAM, the result is a full page fault, exactly as above.

![1751182974723](/assets/images/notes/Linux_virtual_memory_part_1/1751182974723.png)
![1751182986876](/assets/images/notes/Linux_virtual_memory_part_1/1751182986876.png)

**Memory access flow, simplified:**

```
CPU issues virtual address
  -> TLB hit  -> use physical address -> access memory
  -> TLB miss -> is the page in RAM?
                   yes -> walk page table, refill TLB, retry
                   no  -> page fault -> OS swaps the page in -> retry
```

**TLB reach** — the total virtual memory a TLB can cover — scales directly
with page size: 64 entries of 4 KB pages covers 256 KB; 32 entries of 2 MB
pages covers 64 MB.

## Page Faults

A page fault is a processor exception triggered when a program accesses a
virtual page not currently backed by a physical frame in RAM. The CPU
pauses the program, invokes an interrupt service routine (ISR) to locate
and load the missing page, updates the page table, and resumes execution as
if nothing happened.

The eviction step matters for cost: the OS typically picks the least
recently used resident page to evict. If it's dirty (written since it was
loaded), it has to be flushed to disk first; if it's clean, it can just be
dropped, since disk already holds an up-to-date copy — a small but real
performance win for pages that were only ever read. Because disk I/O is so
slow, loading the missing page back in is a good candidate for
[DMA](/articles/linux/dma): letting a DMA controller move the data from
disk to RAM frees the CPU to run a different process in the meantime,
instead of stalling on the transfer.

![1751180496770](/assets/images/notes/Linux_virtual_memory_part_1/1751180496770.png)
![1751180574865](/assets/images/notes/Linux_virtual_memory_part_1/1751180574865.png)
![1751180717922](/assets/images/notes/Linux_virtual_memory_part_1/1751180717922.png)
![1751180729651](/assets/images/notes/Linux_virtual_memory_part_1/1751180729651.png)
![1751180737772](/assets/images/notes/Linux_virtual_memory_part_1/1751180737772.png)
![1751180987788](/assets/images/notes/Linux_virtual_memory_part_1/1751180987788.png)

**Three kinds of page fault:**

| Type | What's happening | Cost |
|---|---|---|
| **Minor (soft)** | The data is already in RAM, just not yet mapped into *this* process's page table — common with shared memory/libraries: the first process to touch a shared library maps it in; every later process touching the same library gets a minor fault, and the OS just adds a mapping to memory that's already resident. | Fast, low impact. |
| **Major (hard)** | The data genuinely isn't in RAM and must be read from disk — typical of demand paging (a page loaded only when first touched) or a page previously swapped out to reclaim RAM. | Slow — can meaningfully degrade performance if frequent. |
| **Invalid** | The accessed address isn't mapped in the process's address space at all, on disk or in RAM. | Fatal — the kernel sends `SIGSEGV` (segmentation fault). |

![1749355062259](/assets/images/notes/Linux_IPC_socket/1749355062259.png)
![1749355098190](/assets/images/notes/Linux_IPC_socket/1749355098190.png)

**Handling a major fault, step by step:**

1. CPU accesses a virtual address; the MMU's page table lookup finds no
   mapping and raises a page fault exception.
2. The OS's page fault handler checks whether the address belongs to a
   valid region (e.g. `malloc`'d or `mmap`'d memory).
3. If valid, it picks a resident page to evict if RAM is full — writing it
   back to disk first only if it's **dirty** (modified since it was loaded;
   a clean page can just be dropped, since a copy already exists on disk).
4. It loads the required page from disk (or zero-fills it, for new
   anonymous memory) into RAM.
5. It updates the page table with the new mapping.
6. Execution resumes at the faulting instruction, which now succeeds.

**Roughly what that costs, in CPU cycles:**

| Step | Approximate cost |
|---|---|
| Check page table | Very fast |
| Raise page fault | 100s–1,000s of cycles |
| Jump to OS handler | ~10,000 cycles |
| Evict a dirty page (write to disk) | ~40,000,000 cycles |
| Read the needed page from disk | ~40,000,000 cycles |
| Update page table | ~1,000 cycles |
| Return to user program | ~10,000 cycles |
| **Total** | **~80,000,000 cycles** |

Disk access is millions of times slower than RAM, which is the entire
reason this number is so large — and it stays large even on SSDs, since the
bottleneck is fundamentally "not RAM," not spinning-disk seek time
specifically. A page fault often costs enough that the OS context-switches
to another process while it waits, rather than burning cycles idle.

Systems differ in how they cope when memory pressure is high: **iOS**
doesn't support paging to disk at all — exceed available memory and the OS
kills the offending app outright, pushing the burden of careful memory use
onto the app. **macOS** (since 10.9) compresses inactive pages in RAM
instead of writing them to disk first — compression is far cheaper than
disk I/O and reduces how often real paging is needed. In general: enough
RAM means paging rarely triggers at all, and adding RAM is a direct,
reliable performance fix for a system that's paging heavily.

![1751778734506](/assets/images/notes/Linux_virtual_memory_part_4/1751778734506.png)
![1751778879980](/assets/images/notes/Linux_virtual_memory_part_4/1751778879980.png)
![1751784202234](/assets/images/notes/Linux_virtual_memory_part_4/1751784202234.png)
![1751784393355](/assets/images/notes/Linux_virtual_memory_part_4/1751784393355.png)

### Inspecting Page Faults & Memory Maps in Practice

Page fault counts are visible per-process via `/proc/[pid]/stat` (`minflt`
= minor faults, `majflt` = major faults) or `vmstat`; frequent major faults
are a real warning sign of thrashing (below).

List every running process with its total/resident/shared page counts, in
MB:

```bash
#!/bin/bash
# Lists all running processes with PID, command, and page counts
# (total, resident, shared), converted from pages to MB.

PAGE_SIZE=$(getconf PAGE_SIZE)  # in bytes

printf "%-8s %-25s %10s %10s %10s\n" "PID" "COMMAND" "TOTAL_PAGES" "RES_PAGES" "SHARED_PAGES"

for pid in $(ls /proc | grep -E '^[0-9]+$'); do
    if [ -r /proc/$pid/statm ] && [ -r /proc/$pid/comm ]; then
        read total resident shared _ < /proc/$pid/statm
        cmd=$(cat /proc/$pid/comm)
        printf "%-8s %-25s %10.2f %10.2f %10.2f\n" \
        "$pid" "$cmd" \
        "$(echo "$total * $PAGE_SIZE / 1048576" | bc -l)" \
        "$(echo "$resident * $PAGE_SIZE / 1048576" | bc -l)" \
        "$(echo "$shared * $PAGE_SIZE / 1048576" | bc -l)"
    fi
done
```

Same idea, but for minor/major fault counts per process:

```bash
#!/bin/bash
# Find out Major and Minor page faults per process running

printf "%-8s %-20s %-10s %-10s\n" "PID" "COMMAND" "MINFLT" "MAJFLT"

for pid in $(ls /proc | grep -E '^[0-9]+$'); do
    if [ -r /proc/$pid/stat ] && [ -r /proc/$pid/comm ]; then
        read minflt _ majflt _ < <(awk '{print $10, $12}' /proc/$pid/stat)
        cmd=$(cat /proc/$pid/comm)
        printf "%-8s %-20s %-10s %-10s\n" "$pid" "$cmd" "$minflt" "$majflt"
    fi
done
```

A self-contained C program that reads its own memory footprint from
`/proc/self/statm` (summary) and `/proc/self/maps` (full mapping detail):

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(void) {
    long total_pages, resident_pages, shared_pages, text, lib, data, dt;
    long page_size = sysconf(_SC_PAGESIZE);

    // --- Part 1: Summary from /proc/self/statm ---
    FILE *fp = fopen("/proc/self/statm", "r");
    if (!fp) {
        perror("fopen(/proc/self/statm)");
        return 1;
    }

    if (fscanf(fp, "%ld %ld %ld %ld %ld %ld %ld",
               &total_pages, &resident_pages, &shared_pages,
               &text, &lib, &data, &dt) != 7) {
        perror("fscanf");
        fclose(fp);
        return 1;
    }
    fclose(fp);

    printf("=== Process Memory Summary (/proc/self/statm) ===\n");
    printf("Page size          : %ld bytes\n", page_size);
    printf("Total pages        : %ld (%.2f KB)\n", total_pages, total_pages * page_size / 1024.0);
    printf("Resident pages     : %ld (%.2f KB)\n", resident_pages, resident_pages * page_size / 1024.0);
    printf("Shared pages       : %ld (%.2f KB)\n", shared_pages, shared_pages * page_size / 1024.0);
    printf("Text (code) pages  : %ld (%.2f KB)\n", text, text * page_size / 1024.0);
    printf("Library pages      : %ld (%.2f KB)\n", lib, lib * page_size / 1024.0);
    printf("Data + Stack pages : %ld (%.2f KB)\n\n", data, data * page_size / 1024.0);

    // --- Part 2: Detailed mappings from /proc/self/maps ---
    printf("=== Detailed Memory Map (/proc/self/maps) ===\n");
    FILE *maps = fopen("/proc/self/maps", "r");
    if (!maps) {
        perror("fopen(/proc/self/maps)");
        return 1;
    }

    char line[512];
    while (fgets(line, sizeof(line), maps)) {
        printf("%s", line);
    }

    fclose(maps);
    return 0;
}
```

Running it **statically linked** (`gcc -static mem_map_info.c -o mem_map_info`)
vs. **dynamically linked** (`gcc mem_map_info.c -o mem_map_info`) makes the
cost of dynamic linking concrete:

```
$ gcc -static mem_map_info.c -o mem_map_info && ls -lh mem_map_info
-rwxrwxr-x 1 ninja ninja 937K Oct 12 01:00 mem_map_info
$ ./mem_map_info
=== Process Memory Summary (/proc/self/statm) ===
Page size          : 4096 bytes
Total pages        : 292 (1168.00 KB)
Resident pages     : 164 (656.00 KB)
Shared pages       : 164 (656.00 KB)
Text (code) pages  : 164 (656.00 KB)
Library pages      : 0 (0.00 KB)
Data + Stack pages : 76 (304.00 KB)
```

```
$ gcc mem_map_info.c -o mem_map_info && ls -lh mem_map_info
-rwxrwxr-x 1 ninja ninja 16K Oct 12 01:11 mem_map_info
$ ./mem_map_info
=== Process Memory Summary (/proc/self/statm) ===
Page size          : 4096 bytes
Total pages        : 671 (2684.00 KB)
Resident pages     : 343 (1372.00 KB)
Shared pages       : 343 (1372.00 KB)
Text (code) pages  : 1 (4.00 KB)
Library pages      : 0 (0.00 KB)
Data + Stack pages : 90 (360.00 KB)
```

**Key observations:**

- The static binary is 937 KB on disk (everything, including libc, baked
  in) but the dynamic one is only 16 KB — the dynamic binary's own code is
  tiny, and `libc.so` / `ld-linux` show up as separate mappings in
  `/proc/self/maps` instead.
- `Total pages > file size` in both cases, because of page alignment plus
  added heap/stack/kernel-provided mappings (`[heap]`, `[stack]`, `[vdso]`,
  `[vvar]`, `[vsyscall]` — the last a legacy syscall page kept for backward
  compatibility, and `[vdso]` a kernel-provided page of fast userspace
  syscalls like `gettimeofday`).
- In the static binary, `shared pages == resident pages` — with no separate
  shared libraries, everything the process maps is technically "shared"
  only with itself.
- `r-xp` mappings are executable code (`.text`), `r--p` is read-only data
  (`.rodata`), `rw-p` is initialized data (`.data`) with a final `rw-p`
  region for zero-filled `.bss`.

## Swapping, Paging & Thrashing

**Paging** is the low-level mechanism: memory is divided into fixed-size
**pages** (virtual side, typically 4 KB) and **frames** (physical side, same
size), with the page table mapping one to the other.

**Swapping** is what that mechanism enables: moving entire processes, or
individual pages, between RAM and disk to free up physical memory for other
work — the basis of running more processes than physical RAM alone could
hold.

| Concept | Description |
|---|---|
| **Paging** | Divides memory into pages and maps them |
| **Swapping** | Moves pages between RAM and disk using that mapping |

A page swap, end to end (**demand paging**): a process accesses a virtual
address → the MMU's page table lookup finds the page isn't resident → page
fault → the OS finds (or evicts) a free frame in RAM, loads the needed page
from swap space, and updates the page table.

**Thrashing** is what happens when a system spends more time swapping pages
than executing actual process code — too many processes competing for too
little RAM, driving a constant stream of page faults:

```
Process A needs page X -> not in RAM -> page fault
OS evicts page Y to make room
Process B needs page Y -> not in RAM -> another page fault
... repeat, indefinitely
```

Too many processes plus too little RAM means constant page faults, and the
system becomes unresponsive — the CPU spends its time handling faults
instead of running application code.

## Memory Protection & Address Space Layout

Each program gets its own virtual address space and page table. Two
processes can use the identical virtual address and still land on entirely
different physical memory — the isolation that keeps one process from
reading or corrupting another's memory, deliberately or by bug.

**A typical (32-bit) Linux process address space layout:**

- **Top 1 GB** — reserved for kernel space, inaccessible to user code.
- **Stack** — grows *downward* from just below kernel space.
- **Heap** — grows *upward*, for dynamic allocation.
- **Libraries** — mapped in the middle (shared code, e.g. libc, UI toolkits).
- **Text segment** — the program's own code.
- **Data segment** — initialized global/static variables.
- **Low ~128 MB** — reserved by the OS (e.g. for I/O memory).

**ASLR (Address Space Layout Randomization)** randomizes the gaps between
these regions (stack, heap, libraries) on each run, making memory locations
harder for an attacker to predict — a meaningful mitigation against a whole
class of memory-corruption exploits.

**Private vs. shared pages:** by default, each process's pages map to
physical memory unique to it (private). Two processes can deliberately
**share** a physical page by mapping their respective virtual pages to the
same physical frame — the basis of shared libraries and some forms of IPC
(e.g. a shared clipboard). The OS switches page tables on every context
switch, so the *same* virtual address in two different processes routinely
points at entirely different physical memory unless they're explicitly
sharing that page.

![1751800942365](/assets/images/notes/Linux_virtual_memory_part_5/1751800942365.png)
![1751801136512](/assets/images/notes/Linux_virtual_memory_part_5/1751801136512.png)
![1751801296347](/assets/images/notes/Linux_virtual_memory_part_5/1751801296347.png)
![1751801377522](/assets/images/notes/Linux_virtual_memory_part_5/1751801377522.png)
![1751801450013](/assets/images/notes/Linux_virtual_memory_part_5/1751801450013.png)
