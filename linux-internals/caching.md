---
layout: default
title: Caching
parent: Linux Internals
nav_order: 3
description: "How CPU caches actually work: locality of reference, cache lines/sets/ways, write policies, dirty bits, and multi-level cache hierarchies."
---

# Caching

## Why the CPU Needs a Cache

Main memory (RAM) is slow relative to the CPU — a single access can cost
hundreds of CPU cycles. If every instruction had to wait on RAM, the
processor would spend most of its time idle instead of computing.

A concrete example of why this matters: a game updating hundreds of on-screen
entities every frame needs to touch each entity's data 60 times a second. If
those entities are stored as separate objects scattered across memory
(a typical object-oriented layout), updating them means jumping to a new,
unrelated memory location for every single entity — expensive when repeated
dozens of times per frame.

![1749402637256](/assets/images/notes/Linux_IPC_socket/1749402637256.png)

Modern engines address this with *data-oriented design*: instead of each
entity owning a scattered blob of properties, all positions are stored
together in one contiguous block, all velocities in another, and so on. This
layout lets the CPU iterate over one tightly-packed array instead of chasing
pointers all over memory — and it's exactly the access pattern a CPU cache is
built to exploit.

![1749402751221](/assets/images/notes/Linux_IPC_socket/1749402751221.png)

The CPU cache is a small, very fast memory that sits between the core and
main memory, holding copies of the data the CPU is likely to need next. The
CPU reads and writes to the cache directly, and only pushes changes back to
main memory later if needed — avoiding a slow round trip to RAM on every
access.

![1749402800318](/assets/images/notes/Linux_IPC_socket/1749402800318.png)

Modern CPUs have multiple cache levels — L1, L2, L3 — of increasing size and
decreasing speed, all located inside or very close to the CPU package. The
rest of this page mostly talks about "the cache" as a single concept before
covering the multi-level hierarchy in detail further down.

![1749402813592](/assets/images/notes/Linux_IPC_socket/1749402813592.png)

## Cache Hits and Misses

When the CPU reads from memory, it first checks whether that address is
already in the cache. The first time a given address is requested, it isn't
in the cache yet — a **cache miss**. On a miss, the data is copied from main
memory into the cache, and so is a run of the addresses immediately
following it.

Because programs tend to access memory sequentially, by the time the next
address in that run is requested, it's already sitting in the cache — a
**cache hit** — and the CPU can read it directly, skipping the slow trip to
main memory entirely.

![1749403702750](/assets/images/notes/Linux_IPC_socket/1749403702750.png)
![1749403758268](/assets/images/notes/Linux_IPC_socket/1749403758268.png)

Storing data contiguously in memory increases the hit rate and cuts down the
time the CPU spends stalled on main memory. This behavior is governed by the
**locality of reference** principle, which has two parts: temporal locality
and spatial locality.

![1749403775107](/assets/images/notes/Linux_IPC_socket/1749403775107.png)

**Temporal locality** — recently accessed data is likely to be accessed
again soon. If a character's position is updated every frame, that address
gets read repeatedly in a short window, so keeping it in the cache avoids
reloading it from main memory each time.

![1749403862113](/assets/images/notes/Linux_IPC_socket/1749403862113.png)
![1749403882385](/assets/images/notes/Linux_IPC_socket/1749403882385.png)

**Spatial locality** — data physically close to an accessed address tends to
be accessed shortly after. Updating one character's position is usually
followed by updating the positions of adjacent characters stored right next
to it.

![1749403910288](/assets/images/notes/Linux_IPC_socket/1749403910288.png)

Organizing data contiguously means that loading one piece of data into the
cache also loads its neighbors — fewer cache misses, more efficient memory
access.

![1749489259343](/assets/images/notes/Linux_IPC_socket/1749489259343.png)

## Cache Structure: Lines, Sets & Ways

A cache organizes its storage into **cache lines** — the smallest chunk of
memory the cache can transfer at once, typically 32, 64, or 128 bytes, with
64 bytes being the most common. Each line is tagged with a number identifying
which region of main memory it came from. To speed up lookups, lines are
grouped into **sets**, and each memory block maps to exactly one set based on
its address — but within that set, it can occupy any of the available lines,
called **ways**. A cache with 8 lines per set is called 8-way set
associative.

- **Set** — a group of cache lines; a memory block maps to a specific set
  based on its address.
- **Way** — a single line within a set; the number of ways is the cache's
  associativity (e.g. a 4-way set associative cache has 4 lines per set).

![1749489579418](/assets/images/notes/Linux_IPC_socket/1749489579418.png)

### Worked Example

Take a computer that can address up to 64 GB of RAM, with a 32 KB cache made
up of 64 sets, 8 ways per set, and 64 bytes per line:

| | |
|---|---|
| Total RAM size | 64 GB |
| Total cache size | 32 KB |
| Sets | 64 |
| Ways (lines/set) | 8 |
| Data bytes/line | 64 bytes |
| Bits to address RAM | 36 |

Those 36 address bits split into three fields:

| Field | Bits | Meaning |
|---|---|---|
| **Tag** | 24 | Identifies 2²⁴ possible regions |
| **Index** | 6 | Selects 1 of 64 sets |
| **Offset** | 6 | Selects 1 of 64 bytes within the line |

Memory is divided into 64-byte blocks (2⁶), so the lowest 6 bits of an
address are the **offset** within a line. The cache has 64 sets (2⁶), so the
next 6 bits are the **set index**. The remaining 24 bits form the **tag**.

When the cache receives an address, it uses the index to pick a set, places
the incoming data in the next free line of that set, and stores the tag
alongside it. On a read, the cache extracts the index from the requested
address, scans the tags of that set's lines for a match, and — on a hit —
uses the offset to pull the exact byte out of the matching line.

[Set Associative Mapping — video walkthrough](https://youtu.be/KhAh6thw_TI?si=Div0hHtchUx9gIOz)

![1749489833836](/assets/images/notes/Linux_IPC_socket/1749489833836.png)
![1749490284917](/assets/images/notes/Linux_IPC_socket/1749490284917.png)

### Associativity Trade-offs

Every address maps to a specific set, but still has some flexibility in
which line within that set it lands on. The two extremes:

- **Fully associative** — a cache with only one set, so data can go
  anywhere. Maximally flexible, but expensive in power and area to search.
- **Direct-mapped** — one way per set, so every address maps to exactly one
  cache location. Very fast to access, but prone to *conflict misses*: if
  the one designated line is already occupied, it must be evicted to make
  room, even if other lines sit empty.

![1749490496617](/assets/images/notes/Linux_IPC_socket/1749490496617.png)
![1749490563783](/assets/images/notes/Linux_IPC_socket/1749490563783.png)

Most real caches sit between these two extremes as N-way set associative
caches. Because the cache is small relative to main memory, it fills up
quickly and has to evict something to make room for new data — decided by a
**cache replacement algorithm** (LRU and its approximations are the most
common; a deeper look at replacement policies is a topic on its own).

![1749490608390](/assets/images/notes/Linux_IPC_socket/1749490608390.png)

## Cache Coherence & Write Policies

Reads are simple: the CPU always gets the correct, up-to-date data for a
given address, whether it's a hit or a miss. Writes are where it gets
interesting. When the CPU modifies data in the cache, main memory still
holds the old value at that address — the **cache coherence problem**. If
another component reads that address directly from memory, it gets stale
data.

![1749732588843](/assets/images/notes/Linux_Cache_part-2/1749732588843.png)
![1749732622020](/assets/images/notes/Linux_Cache_part-2/1749732622020.png)
![1749732652897](/assets/images/notes/Linux_Cache_part-2/1749732652897.png)
![1749732663422](/assets/images/notes/Linux_Cache_part-2/1749732663422.png)

This gets more serious in multicore/multiprocessor systems, where every core
has its own cache but all of them share the same main memory: if one core
modifies its cached copy without telling the others, the rest keep reading
stale data from memory or from their own caches.

![1749732719512](/assets/images/notes/Linux_Cache_part-2/1749732719512.png)

Write policies cover two independent decisions: what happens on a **write
hit**, and what happens on a **write miss**.

![1749732837591](/assets/images/notes/Linux_Cache_part-2/1749732837591.png)

### Write Hits: Write-Through vs. Write-Back

On a write hit, the address is already in the cache, and the cache
controller updates it there — which immediately puts the cache and main
memory out of sync. Two policies resolve this:

**Write-through** — every write to the cache is immediately propagated to
main memory too, keeping the two always in sync. This simplifies tracking
modified data, but every write now pays for two updates instead of one,
slowing writes down and adding traffic to the memory bus.

![1749732982758](/assets/images/notes/Linux_Cache_part-2/1749732982758.png)
![1749733307885](/assets/images/notes/Linux_Cache_part-2/1749733307885.png)

**Write-back** — a write updates only the cache; main memory is updated
later, typically on eviction or an explicit flush. Multiple writes to the
same line collapse into a single memory write, so this policy is faster and
uses less bus bandwidth — at the cost of more complex bookkeeping, and a
real risk of data loss if power is lost before a modified line is written
back.

![1749732966549](/assets/images/notes/Linux_Cache_part-2/1749732966549.png)
![1749733334804](/assets/images/notes/Linux_Cache_part-2/1749733334804.png)
![1749733364686](/assets/images/notes/Linux_Cache_part-2/1749733364686.png)

Write-back caches track this with a **dirty bit** per cache line: set to 1
whenever the line is modified, meaning the cache copy no longer matches main
memory. On eviction, the controller checks the dirty bit — if it's 0, the
line is discarded immediately (cache and memory already agree); if it's 1,
the modified data is written back to main memory first. A cache flush walks
every line, writes back anything with the dirty bit set, then clears it.

![1749733400507](/assets/images/notes/Linux_Cache_part-2/1749733400507.png)

### Write Misses: Write-Allocate vs. No-Write-Allocate

On a write miss, the address being written isn't in the cache. Two policies:

**Write-allocate** — a line is allocated in the cache, the data is loaded
from main memory, and the write is applied there. If paired with
write-through, the update also goes to memory immediately; if paired with
write-back, the dirty bit is set and memory is updated later. This assumes
the newly written data will be accessed again soon — a good bet for data
that gets reused, but wasteful (and a source of *cache pollution*, where
rarely-used data displaces useful data) if the address is written once and
never touched again.

![1749734541053](/assets/images/notes/Linux_Cache_part-2/1749734541053.png)

**No-write-allocate** — the write bypasses the cache entirely and goes
straight to main memory. This avoids the overhead of loading data into the
cache just to write it once, and avoids polluting the cache with data that
won't be reused — at the cost of a guaranteed miss if that address *is* read
again soon.

**Summary:**

| | Write Hit | Write Miss |
|---|---|---|
| **Write-through** | Update cache → update RAM immediately | — |
| **Write-back** | Update cache → update RAM later (on eviction/flush) | — |
| **Write-allocate** | — | Load line into cache → update → write back later |
| **No-write-allocate** | — | Write directly to RAM, bypassing the cache |

![1749734600824](/assets/images/notes/Linux_Cache_part-2/1749734600824.png)

In practice, **write-back + write-allocate** is the common pairing for CPU
L1 data caches, since both policies maximize cache utilization and minimize
memory bus traffic for hot, frequently-reused data. **Write-through +
no-write-allocate** pairs well for workloads like logging, where data is
written once and rarely read again — no benefit to caching it, and this
combination avoids polluting the cache with it. Other combinations exist for
other trade-offs, but these two are the common ones.

![1749737652003](/assets/images/notes/Linux_Cache_part-2/1749737652003.png)

## Multi-Level Cache Hierarchies

Modern CPUs use a hierarchy of caches: the level closest to the core is the
smallest and fastest, and each level further out is larger, slower, and more
complex.

![1749882390239](/assets/images/notes/Linux_Cache_Hierarchy/1749882390239.png)
![1749882654008](/assets/images/notes/Linux_Cache_Hierarchy/1749882654008.png)
![1749882681542](/assets/images/notes/Linux_Cache_Hierarchy/1749882681542.png)

**L1** is kept very small to match the speed of the core, and is typically
split into separate instruction and data caches. When L1's small size
becomes a bottleneck, **L2** — larger, slower, usually unified for both data
and instructions, and dedicated to a single core — sits behind it and talks
directly to L1.

![1749882930532](/assets/images/notes/Linux_Cache_Hierarchy/1749882930532.png)

On multi-core systems, an **L3** cache is typically shared across all cores:
larger and slower than L2, it lets cores share data without going all the
way to main memory, and is checked when both L1 and L2 miss. Some
specialized systems add an L4 on top of L1–L3 for even more headroom.

![1749883099931](/assets/images/notes/Linux_Cache_Hierarchy/1749883099931.png)

Typical numbers per level:

| Level | Size (per core) | Associativity | Latency |
|---|---|---|---|
| **L1** | 16 KB – 128 KB | 2–8 ways | Few CPU cycles |
| **L2** | 256 KB – 2 MB (up to several MB on older CPUs) | 4–16 ways | 4–10 cycles |
| **L3** | 2 MB – 32+ MB (shared) | ~16 ways (varies) | 10–40 cycles |

![1749883138769](/assets/images/notes/Linux_Cache_Hierarchy/1749883138769.png)
![1749883180739](/assets/images/notes/Linux_Cache_Hierarchy/1749883180739.png)
![1749883220610](/assets/images/notes/Linux_Cache_Hierarchy/1749883220610.png)

### Inclusion Policies

Inclusion policies decide whether a data block can live in just one cache
level, be copied across multiple levels, or something more flexible:

![1749883272767](/assets/images/notes/Linux_Cache_Hierarchy/1749883272767.png)

- **Inclusive** — data in a higher level (e.g. L1) is also duplicated in the
  levels below it (L2, and possibly L3). L2 is said to *include* L1.
  ![1749883305707](/assets/images/notes/Linux_Cache_Hierarchy/1749883305707.png)
- **Exclusive** — a block exists in exactly one cache level at a time. If
  it's in L1, it's not in L2 or L3. L2 is *exclusive* of L1.
  ![1749883376282](/assets/images/notes/Linux_Cache_Hierarchy/1749883376282.png)
- **Non-inclusive, non-exclusive** — a hybrid with no strict duplication
  rule; a block may or may not appear in multiple levels depending on system
  design.
  ![1749883401054](/assets/images/notes/Linux_Cache_Hierarchy/1749883401054.png)

Real CPUs mix policies across levels — for example, Intel's Sandy Bridge,
Ivy Bridge, and Skylake pair an inclusive L3 with a non-inclusive,
non-exclusive L2.

![1749883438464](/assets/images/notes/Linux_Cache_Hierarchy/1749883438464.png)

### Reads and Writes Through the Hierarchy

**Read**, in a fully inclusive three-level hierarchy: check L1 first — hit,
and the data goes straight to the core. Miss, and check L2 — hit, and the
data is copied into L1 *and* forwarded to the core (so a repeat access hits
in L1). Miss again, and check L3 the same way, copying down through L2 into
L1 on a hit. If all three miss, the data comes from main memory and is
copied into L3, L2, and L1 on its way to the core.

![1749883557741](/assets/images/notes/Linux_Cache_Hierarchy/1749883557741.png)
![1749883577145](/assets/images/notes/Linux_Cache_Hierarchy/1749883577145.png)
![1749883586432](/assets/images/notes/Linux_Cache_Hierarchy/1749883586432.png)
![1749883616860](/assets/images/notes/Linux_Cache_Hierarchy/1749883616860.png)

**Write** behavior (assuming all levels share the same policy for
simplicity): under write-through, a write to L1 immediately propagates to
L2, L3, and main memory, keeping every level in sync.

![1749883745091](/assets/images/notes/Linux_Cache_Hierarchy/1749883745091.png)

Under write-back, only L1 is updated immediately and marked dirty; lower
levels are updated on eviction. When a dirty L1 line is evicted, it's
written to L2 (marked dirty there too), and so on down the hierarchy.

![1749883771978](/assets/images/notes/Linux_Cache_Hierarchy/1749883771978.png)
![1749883790182](/assets/images/notes/Linux_Cache_Hierarchy/1749883790182.png)

Write misses follow the same allocate / no-allocate split as a single-level
cache, applied at whichever level takes the miss: write-allocate brings the
block into the hierarchy and updates it there; no-write-allocate bypasses
the cache and writes straight to the next level down — all the way to main
memory if every level is configured no-write-allocate.

![1749883905283](/assets/images/notes/Linux_Cache_Hierarchy/1749883905283.png)
![1749883925965](/assets/images/notes/Linux_Cache_Hierarchy/1749883925965.png)

## Summary

- Caches exist because main memory is slow; locality of reference (temporal
  + spatial) is why they work.
- A cache is organized into lines, grouped into sets; associativity (ways
  per set) trades off flexibility against lookup cost — direct-mapped and
  fully associative are the two extremes, N-way set associative is the
  practical middle ground.
- Write hits are handled by write-through (always sync to memory) or
  write-back (sync lazily, tracked via a dirty bit); write misses are
  handled by write-allocate (load into cache, then write) or
  no-write-allocate (bypass the cache). Write-back+write-allocate suits
  hot, reused data; write-through+no-write-allocate suits write-once data.
- Real CPUs stack L1 (per-core, split I/D, smallest/fastest) → L2
  (per-core, unified) → L3 (shared, largest/slowest), governed by an
  inclusion policy (inclusive, exclusive, or non-inclusive/non-exclusive)
  that decides how data is duplicated across levels.
