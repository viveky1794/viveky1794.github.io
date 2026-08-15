---
layout: default
title: Scheduling
parent: Linux Internals
nav_order: 5
description: "CPU scheduling from first principles: PCBs, the scheduler vs. the dispatcher, CPU/IO bursts, FCFS, SJF, round robin, priority scheduling, aging, and multilevel feedback queues."
---

# CPU Scheduling

Even when a benchmark pins CPU usage at 100%, a general-purpose OS still
feels responsive — because the scheduler prioritizes response time for
interactive work even under heavy load. CPU scheduling is the technique an
OS uses to decide which process gets the CPU at any given moment, based on
criteria like response time, CPU utilization, throughput, turnaround time,
and waiting time — all covered below.

![1750700100482](/Linux/Core/image/Linux_scheduler/1750700100482.png)
![1750700399476](/Linux/Core/image/Linux_scheduler/1750700399476.png)

## The Process Control Block and "Allocating the CPU"

A process's memory includes a **text section** holding its executable code.
The CPU has an **address register** (program counter) that points to the
next instruction to execute. "Allocating the CPU to a process" means
pointing that register at the process's text section; reallocating it to a
different process means updating the register to point elsewhere — a
**context switch**.

![1750700494409](/Linux/Core/image/Linux_scheduler/1750700494409.png)
![1750700521607](/Linux/Core/image/Linux_scheduler/1750700521607.png)
![1750700531899](/Linux/Core/image/Linux_scheduler/1750700531899.png)
![1750700542664](/Linux/Core/image/Linux_scheduler/1750700542664.png)
![1750700592102](/Linux/Core/image/Linux_scheduler/1750700592102.png)
![1750700605014](/Linux/Core/image/Linux_scheduler/1750700605014.png)

A process isn't a concrete object — it's a context the computer operates
in. The OS represents that context with a **process control block (PCB)**:
a struct holding the process's state, program counter, registers, and other
management data. One is created whenever a process is created.

![1750700825235](/Linux/Core/image/Linux_scheduler/1750700825235.png)
![1750700852767](/Linux/Core/image/Linux_scheduler/1750700852767.png)

Since a process itself can't be pushed into a queue, what actually moves
through scheduling queues is each process's PCB — commonly just called "the
process" for short, even though it's really a reference to it.

![1750700919373](/Linux/Core/image/Linux_scheduler/1750700919373.png)
![1750700935002](/Linux/Core/image/Linux_scheduler/1750700935002.png)
![1750700949740](/Linux/Core/image/Linux_scheduler/1750700949740.png)
![1750701101116](/Linux/Core/image/Linux_scheduler/1750701101116.png)

## Why "Run Until It Finishes" Doesn't Work

The naive model — the CPU runs one process to completion before starting
the next — breaks for two reasons:

1. **Many processes never terminate.** Servers and background services run
   indefinitely; if one holds the CPU forever, everything behind it in the
   queue starves.
2. **Processes don't use the CPU continuously.** A task that takes 2 minutes
   wall-clock doesn't hold the CPU for all 2 minutes — it spends much of
   that time waiting on something else, typically I/O.

![1750701154659](/Linux/Core/image/Linux_scheduler/1750701154659.png)
![1750701225653](/Linux/Core/image/Linux_scheduler/1750701225653.png)
![1750701283835](/Linux/Core/image/Linux_scheduler/1750701283835.png)
![1750701318888](/Linux/Core/image/Linux_scheduler/1750701318888.png)

Take a program copying a file 8 bytes at a time.

![1750783113571](/Linux/Core/image/Linux_scheduler/1750783113571.png)
![1750783130931](/Linux/Core/image/Linux_scheduler/1750783130931.png)

The instructions look sequential, but reads and writes are system calls that
hand off to the disk — an I/O operation the CPU can't just push through:

![1750783143732](/Linux/Core/image/Linux_scheduler/1750783143732.png)

Those calls can't complete until the I/O hardware finishes, independent of
CPU speed.

![1750783243450](/Linux/Core/image/Linux_scheduler/1750783243450.png)

So the program's CPU usage over time isn't continuous — it has gaps where
the CPU sits idle waiting on I/O.

![1750783308030](/Linux/Core/image/Linux_scheduler/1750783308030.png)

This is the norm, not the exception, on virtually all modern (Von Neumann)
computers, and it's been studied for decades. The periods a process spends
actually using the CPU are **CPU bursts**; the periods it spends waiting on
I/O are **I/O bursts**. Every process execution alternates: CPU burst → I/O
burst → CPU burst → ... → a final CPU burst that ends in a termination
request.

![1750783381978](/Linux/Core/image/Linux_scheduler/1750783381978.png)

Measured CPU burst durations across real systems follow a predictable
distribution: most processes generate a large number of short CPU bursts and
only a small number of long ones. This single observation drives most of CPU
scheduler design — a scheduler that ignores it wastes a lot of CPU time.

![1750783497381](/Linux/Core/image/Linux_scheduler/1750783497381.png)

The goal, then: when a process enters an I/O burst, hand the CPU to a
*different* process, ideally one not also about to block on I/O. Note also
that a burst never starts immediately after the previous one ends — there's
always a small extra gap (dispatch latency, covered below).

![1750783677914](/Linux/Core/image/Linux_scheduler/1750783677914.png)
![1750783653680](/Linux/Core/image/Linux_scheduler/1750783653680.png)
![1750784870630](/Linux/Core/image/Linux_scheduler/1750784870630.png)

A scheduling algorithm therefore has to allocate the CPU only until a
process's *current* CPU burst ends, not until the process fully terminates —
which means tracking several distinct process states.

## Process States

- **New** — the process has just launched; its executable is still being
  loaded.
- **Ready** — loaded and set up, waiting for CPU time.
- **Running** — the CPU is allocated to it.
- **Waiting** — blocked on an I/O request or some other event (a keystroke,
  a network packet), having voluntarily given up the CPU.
- **Terminated** — done executing.

A process cycles between ready, running, and waiting for most of its
lifetime; some (web servers) never leave that cycle. When a waiting I/O
operation completes, the process goes back to *ready*, not straight to
*running* — the CPU has likely been reallocated elsewhere in the meantime,
so it still has to wait its turn.

These state names are conventional and vary by OS, though the underlying
states are universal; some systems also distinguish a process that's *fully*
terminated from one still in the process of releasing its resources (memory,
etc.) after an exit call.

![1750788229806](/Linux/Core/image/Linux_scheduler/1750788229806.png)

## Scheduler vs. Dispatcher

This state model is why queues are the fundamental data structure in CPU
scheduling: at any moment hundreds of processes might be in a non-running
state, and only one process can run on a core at a time — so "ready to run"
processes queue up, but "the currently running process" doesn't belong in
any queue.

![1750788266133](/Linux/Core/image/Linux_scheduler/1750788266133.png)

The **dispatcher** is the component that actually performs the context
switch: when the running process blocks on I/O (or the CPU otherwise becomes
free), the dispatcher allocates the CPU to the process at the head of the
ready queue.

![1750788382287](/Linux/Core/image/Linux_scheduler/1750788382287.png)

Concretely, the dispatcher: saves the CPU state of the interrupted process
into its PCB, puts that PCB on the waiting queue, retrieves the PCB of the
next ready process, restores its saved CPU state (registers, program
counter), and reallocates the CPU — plus switches to user mode if the
processor supports it.

![1750788428592](/Linux/Core/image/Linux_scheduler/1750788428592.png)

In short: the **scheduler** decides *which* process runs next by managing
the ready queue according to some policy; the **dispatcher** *carries out*
that decision. Because the dispatcher itself consumes CPU time to run, every
context switch introduces unavoidable overhead — **dispatch latency**, the
time to stop one process and start the next. It's usually left off Gantt
charts (the bar-chart convention used throughout this page to show process
schedules) since it's implicit, but it's exactly why the dispatcher is one
of the most heavily optimized parts of an OS.

![1750788805700](/Linux/Core/image/Linux_scheduler/1750788805700.png)
![1750788784578](/Linux/Core/image/Linux_scheduler/1750788784578.png)

## First-Come, First-Served (FCFS)

The simplest scheduling algorithm: a FIFO ready queue. When a process is
created its PCB goes on the tail; when the CPU frees up, it's allocated to
the PCB at the head — and as soon as that process makes an I/O request, the
CPU moves to the next one in line. Strictly, this schedules *CPU bursts*,
not whole processes, so "first CPU burst come, first served" would be more
precise — but "FCFS" is the name everyone uses.

![1750789066799](/Linux/Core/image/Linux_scheduler/1750789066799.png)
![1750788995367](/Linux/Core/image/Linux_scheduler/1750788995367.png)

FCFS does account for CPU utilization, but doesn't optimize it. Processes
with mostly short CPU bursts are **I/O-bound** (their performance improves
with faster I/O); processes with long CPU bursts are **CPU-bound** (they
need a faster CPU, not faster I/O). Long CPU bursts are rare but never
impossible — and that's where FCFS falls apart.

![1750789448340](/Linux/Core/image/Linux_scheduler/1750789448340.png)
![1750789541698](/Linux/Core/image/Linux_scheduler/1750789541698.png)
![1750789553651](/Linux/Core/image/Linux_scheduler/1750789553651.png)

**The convoy effect:** with one CPU-bound process and several I/O-bound
ones, the CPU-bound process grabs the CPU and holds it for a long stretch.
Meanwhile every I/O-bound process finishes its I/O and piles up in the ready
queue — with the I/O devices now sitting idle. When the CPU-bound process
finally releases the CPU, all the I/O-bound processes run their short bursts
quickly and go straight back to I/O, leaving the CPU idle again until the
CPU-bound process returns to the ready queue and repeats the cycle. Every
short process ends up waiting behind the one long one — lower CPU *and*
device utilization than necessary.

![1750856106052](/Linux/Core/image/Linux_scheduler/1750856106052.png)

## Shortest Job First (SJF)

**Shortest Job First** (more precisely: shortest *next CPU burst* first)
allocates the CPU to whichever ready process has the shortest next burst,
using FCFS as a tiebreaker. Implemented as a priority queue keyed on next
burst length, it lets short bursts overtake CPU-hungry ones, cutting the
convoy effect — and it's provably optimal, giving the minimum possible
average waiting time for a given set of processes.

![1750856293715](/Linux/Core/image/Linux_scheduler/1750856293715.png)

With four processes arriving together — one needing 2 minutes, the rest
just milliseconds — FCFS would stall the short ones behind the long one.
Running the short ones first costs the long process only a few extra
milliseconds' delay, while saving the short ones from a multi-minute wait:
moving a short process ahead of a long one reduces the short process's
waiting time by more than it increases the long process's, so average
waiting time drops.

![1750856683021](/Linux/Core/image/Linux_scheduler/1750856683021.png)
![1750856702890](/Linux/Core/image/Linux_scheduler/1750856702890.png)
![1750856729236](/Linux/Core/image/Linux_scheduler/1750856729236.png)

**The catch:** SJF requires knowing the length of a process's next CPU
burst in advance, which is impossible without seeing the future — loops,
function calls, and external input (like user keystrokes) all make exact
prediction unreliable. In practice, SJF *estimates* the next burst from the
process's own history, using an exponential average of previously observed
bursts:

![1750856814307](/Linux/Core/image/Linux_scheduler/1750856814307.png)
![1750856844037](/Linux/Core/image/Linux_scheduler/1750856844037.png)

Each process's prediction depends only on its own burst history — burst *n*
is the most recent observed burst, *n−1* the one before it, and so on;
*n+1* is the burst being predicted.

![1750856880060](/Linux/Core/image/Linux_scheduler/1750856880060.png)

The formula lets the current prediction expand recursively into all past
predictions, which means the scheduler only needs to keep the single most
recent estimate — not a full history of past burst lengths.

![1750857057000](/Linux/Core/image/Linux_scheduler/1750857057000.png)
![1750857089504](/Linux/Core/image/Linux_scheduler/1750857089504.png)

The weighting factor **α** controls how much the estimate trusts recent
history vs. the past: α = 0 ignores the latest burst entirely (prediction
relies only on history); α = 1 ignores all history (prediction is just the
latest burst). α = 0.5 is a common middle ground. Expanding the recursion
shows each past burst is weighted by α(1 − α)ⁿ, an exponentially decaying
term — older bursts matter less, which matches intuition. A process with no
history yet (just created) starts from a constant default, typically a
system-wide average.

![1750857133451](/Linux/Core/image/Linux_scheduler/1750857133451.png)
![1750857183258](/Linux/Core/image/Linux_scheduler/1750857183258.png)
![1750857200586](/Linux/Core/image/Linux_scheduler/1750857200586.png)
![1750857244159](/Linux/Core/image/Linux_scheduler/1750857244159.png)
![1750857250814](/Linux/Core/image/Linux_scheduler/1750857250814.png)
![1750857258728](/Linux/Core/image/Linux_scheduler/1750857258728.png)
![1750857289182](/Linux/Core/image/Linux_scheduler/1750857289182.png)
![1750857299784](/Linux/Core/image/Linux_scheduler/1750857299784.png)

## Preemption and Starvation

If process A is running and a shorter-burst process B enters the ready
queue, should A be interrupted for B? A **preemptive** scheduler can
interrupt a running process; a **non-preemptive** one never does — B waits
regardless of how much shorter its burst is.

![1750857724984](/Linux/Core/image/Linux_scheduler/1750857724984.png)

Preemption is the default in modern general-purpose systems for a concrete
reason: in a non-preemptive system, a process with an extremely long (or
infinite — think a loop with no system call inside it) next burst simply
freezes everything behind it, since nothing can force it off the CPU. Given
that concurrency depends on the illusion of processes progressing
simultaneously, one badly-behaved (or malicious) process holding the CPU
indefinitely breaks that illusion for everyone.

![1750857779543](/Linux/Core/image/Linux_scheduler/1750857779543.png)

Any scheduling policy that leaves a process waiting indefinitely in the
ready queue causes **starvation**. SJF is itself vulnerable to it: a process
with a long estimated burst can be pushed back forever by a steady stream of
shorter-burst arrivals — it would only get the CPU if the ready queue ever
happened to empty out, unlikely on a real system with hundreds of competing
processes. Starvation is a property of the scheduling *policy*, not of any
individual process.

![1750860135509](/Linux/Core/image/Linux_scheduler/1750860135509.png)
![1750860161686](/Linux/Core/image/Linux_scheduler/1750860161686.png)

## Round Robin

Round robin is FCFS plus preemption: a FIFO ready queue, but with a fixed
**time quantum** (e.g. 100 ms) capping how long any process can run before
being forced off. New processes join the tail. The scheduler dispatches the
head of the queue with a timer set for one quantum. Either the process's
burst finishes before the timer fires (it releases the CPU voluntarily, and
the scheduler moves to the next process), or the timer fires first (forcing
a context switch, with the interrupted process going to the tail of the
queue). No process can hold the CPU indefinitely — which directly prevents
the starvation round robin would otherwise be vulnerable to.

![1750869655316](/Linux/Core/image/Linux_scheduler/1750869655316.png)
![1750869666919](/Linux/Core/image/Linux_scheduler/1750869666919.png)
![1750869727089](/Linux/Core/image/Linux_scheduler/1750869727089.png)
![1750869746226](/Linux/Core/image/Linux_scheduler/1750869746226.png)
![1750869764372](/Linux/Core/image/Linux_scheduler/1750869764372.png)
![1750869814785](/Linux/Core/image/Linux_scheduler/1750869814785.png)
![1750869822577](/Linux/Core/image/Linux_scheduler/1750869822577.png)
![1750869851015](/Linux/Core/image/Linux_scheduler/1750869851015.png)
![1750869870434](/Linux/Core/image/Linux_scheduler/1750869870434.png)

Preemption depends on hardware support: the timer is a physical device,
typically embedded in the CPU itself.

If a process's burst finishes without an I/O request, it re-enters the ready
state directly, waiting for its next quantum turn. Round robin distributes
CPU time fairly: with *n* processes and quantum *q*, each process gets
roughly 1/*n* of the CPU in chunks of at most *q*, and waits at most
(*n*−1)×*q* between turns. But fairness isn't the same as optimal waiting
time — with too large a quantum, most processes finish their burst (or hit
I/O) before being preempted, and round robin degenerates into FCFS.

![1750871016722](/Linux/Core/image/Linux_scheduler/1750871016722.png)

Shrinking the quantum doesn't just fix that for free, though: too small a
quantum causes excessive context switches, and context-switch time is pure
overhead — the system does no useful work while switching. The CPU can end
up spending more time switching between processes than running them.

![1750871364639](/Linux/Core/image/Linux_scheduler/1750871364639.png)
![1750871375377](/Linux/Core/image/Linux_scheduler/1750871375377.png)
![1750871700917](/Linux/Core/image/Linux_scheduler/1750871700917.png)
![1750871714428](/Linux/Core/image/Linux_scheduler/1750871714428.png)
![1750871739472](/Linux/Core/image/Linux_scheduler/1750871739472.png)

## Scheduling Metrics

- **Turnaround time** — total time from a process's creation to its
  completion: time waiting in the ready queue + time executing + time doing
  I/O.
- **Throughput** — processes completed per unit time. (A quantum shorter
  than the average context-switch cost tanks this: the CPU stays "busy" but
  mostly switching, not executing.)
- **Response time** — time from a request being submitted to its *first*
  response, not full completion. This is what interactive systems actually
  live or die by, since a process can be producing useful output long before
  it terminates (and some processes never terminate at all).

![1750871780200](/Linux/Core/image/Linux_scheduler/1750871780200.png)
![1750871921097](/Linux/Core/image/Linux_scheduler/1750871921097.png)
![1750871944464](/Linux/Core/image/Linux_scheduler/1750871944464.png)

Average turnaround time doesn't simply improve as the quantum grows — it's
generally best when most processes finish their next burst inside a single
quantum, so context switches only happen when they're actually needed.

Round robin's fairness also has a scaling limit: as the number of
concurrent processes grows, each one waits longer for its turn, and the
"simultaneous execution" illusion breaks down as response time climbs.
Shrinking the quantum to compensate just reintroduces the context-switch
overhead problem. Beyond raw hardware upgrades (faster CPU, more cores),
the software fix is a smarter scheduler.

## Priority Scheduling and Aging

Not every process deserves equal CPU time — a foreground game competes with
a mail client, a file explorer, and background services that barely matter
to the user's experience moment-to-moment. **Priority scheduling** assigns
each process an integer priority (the range and which direction — low
number = high priority or the reverse — vary by OS) and services higher
priority first via a priority queue.

![1750872178841](/Linux/Core/image/Linux_scheduler/1750872178841.png)
![1750872359699](/Linux/Core/image/Linux_scheduler/1750872359699.png)
![1750872377651](/Linux/Core/image/Linux_scheduler/1750872377651.png)
![1750872386707](/Linux/Core/image/Linux_scheduler/1750872386707.png)
![1750872395189](/Linux/Core/image/Linux_scheduler/1750872395189.png)
![1750872404060](/Linux/Core/image/Linux_scheduler/1750872404060.png)
![1750872415137](/Linux/Core/image/Linux_scheduler/1750872415137.png)

It's commonly combined with round robin for fairness among same-priority
processes. Priority scheduling inherits SJF's starvation problem (SJF is,
in fact, just priority scheduling where priority = inverse of the predicted
next burst) — a steady stream of high-priority arrivals can starve a
low-priority process indefinitely. The canonical example: an IBM 7094 at
NASA reportedly ran from 1962 to 1973 with a low-priority job submitted in
1967 that never got to run before shutdown.

![1750872673767](/Linux/Core/image/Linux_scheduler/1750872673767.png)
![1750872709816](/Linux/Core/image/Linux_scheduler/1750872709816.png)
![1750872719887](/Linux/Core/image/Linux_scheduler/1750872719887.png)

**Aging** fixes this by gradually raising the priority of a process the
longer it waits — e.g. bumping a process's priority by one every second it
sits in the queue — until even a process that started at the lowest
priority eventually becomes the highest-priority process in the system and
runs.

## Multilevel Queue and Multilevel Feedback Queue

Combining priority and round robin at scale, a single priority queue can
require an O(n) scan to find the highest-priority process; it's often
simpler to keep **separate queues per priority level** — multilevel queue
scheduling. One approach gives each queue absolute priority over lower
ones (only run a lower queue when all higher ones are empty); another
time-slices across queues in a round-robin fashion, weighted by priority, to
avoid starving lower queues outright. This also lets each queue run its own
scheduling algorithm — e.g. round robin for a foreground/interactive queue,
FCFS for a background queue — matching different classes of processes to
different responsiveness needs.

![1750872792963](/Linux/Core/image/Linux_scheduler/1750872792963.png)
![1750872804279](/Linux/Core/image/Linux_scheduler/1750872804279.png)
![1750872816897](/Linux/Core/image/Linux_scheduler/1750872816897.png)
![1750872845011](/Linux/Core/image/Linux_scheduler/1750872845011.png)
![1750872853465](/Linux/Core/image/Linux_scheduler/1750872853465.png)
![1750872879463](/Linux/Core/image/Linux_scheduler/1750872879463.png)
![1750873195811](/Linux/Core/image/Linux_scheduler/1750873195811.png)

The gap: in plain multilevel queue scheduling, a process is permanently
assigned to a queue on entry — low overhead, but inflexible, since a
process's CPU usage pattern often changes over its lifetime (a mail client
idling in the background vs. suddenly needing to feel instant once brought
to the foreground).

**Multilevel feedback queue** scheduling adapts to that: every process
starts in the highest-priority queue. Finish a burst within one quantum, and
it stays there (or returns there after I/O); need more time than the
quantum allows, and it gets demoted to a lower-priority queue — which has a
*larger* quantum, giving CPU-bound work bigger uninterrupted slices while
still capping its priority. A process can also move back up if it starts
behaving like an I/O-bound task again. Over time, short-burst processes
settle near the top and CPU-bound ones sink — the system infers process
behavior automatically instead of requiring it to be declared up front.

![1750873264570](/Linux/Core/image/Linux_scheduler/1750873264570.png)
![1750873279141](/Linux/Core/image/Linux_scheduler/1750873279141.png)
![1750873286701](/Linux/Core/image/Linux_scheduler/1750873286701.png)
![1750873330833](/Linux/Core/image/Linux_scheduler/1750873330833.png)
![1750873338252](/Linux/Core/image/Linux_scheduler/1750873338252.png)
![1750873351149](/Linux/Core/image/Linux_scheduler/1750873351149.png)
![1750873369330](/Linux/Core/image/Linux_scheduler/1750873369330.png)
![1750873381731](/Linux/Core/image/Linux_scheduler/1750873381731.png)
![1750873406832](/Linux/Core/image/Linux_scheduler/1750873406832.png)
![1750873508124](/Linux/Core/image/Linux_scheduler/1750873508124.png)
![1750873543675](/Linux/Core/image/Linux_scheduler/1750873543675.png)

This is one configuration among many — the demotion/promotion criteria can
use predicted (not just most-recent) burst length to avoid overreacting to
one unusual burst, for example.

![1750873583384](/Linux/Core/image/Linux_scheduler/1750873583384.png)

**Does demoting CPU-bound processes contradict giving important processes
priority?** Not really, for two reasons: I/O-bound and interactive processes
aren't identical, but both are characterized by short bursts; and modern
CPUs are fast enough that even with hundreds of processes running, combined
CPU usage is often under 10%. So higher-priority queues are frequently
empty — a process that's sunk to the lowest queue still gets dispatched
fairly often without needing aging to rescue it. This is exactly why a
system under heavy CPU load (video rendering, a benchmark) still keeps
music playback and browser interaction smooth: everyday interactive tasks
keep top priority, and the CPU-heavy background job gets whatever cycles
are left over.

![1750873861292](/Linux/Core/image/Linux_scheduler/1750873861292.png)
![1750873890013](/Linux/Core/image/Linux_scheduler/1750873890013.png)
![1750873919454](/Linux/Core/image/Linux_scheduler/1750873919454.png)
![1750874053164](/Linux/Core/image/Linux_scheduler/1750874053164.png)

Multilevel feedback queue (in some variant) was the historical default
across many general-purpose operating systems, though modern systems add
further complexity on top of it (multi-core topology, thread priorities,
power/thermal constraints) — worth its own dedicated page down the line.

## Closing Notes

- Modern systems schedule **threads**, not processes — which is how a
  CPU-heavy application can keep its UI thread responsive independently of
  a background compute thread, even under a multilevel feedback queue.
- The I/O queue shown throughout this page is simplified as FIFO for
  clarity. Real I/O subsystems reorder requests (a process waiting on a GPU
  shouldn't queue behind one waiting on disk) — that reordering is the job
  of a separate component, the **I/O scheduler**.

![1750874095695](/Linux/Core/image/Linux_scheduler/1750874095695.png)
![1750874101909](/Linux/Core/image/Linux_scheduler/1750874101909.png)
![1750874135331](/Linux/Core/image/Linux_scheduler/1750874135331.png)
