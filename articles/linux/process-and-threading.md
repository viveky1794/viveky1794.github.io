---
layout: default
title: Process & Threading
parent: Linux Internals
grand_parent: Articles
nav_order: 4
description: "Processes, the PCB, context switching, threads, and how threads behave on single-core vs multi-core systems."
redirect_from:
  - /linux-internals/process-and-threading/
  - /linux-internals/process-and-threading.html
---

# Process & Threading

A process is a running program plus everything the OS needs to manage it —
its own address space, open files, and CPU state. A thread is a lighter-weight
unit of execution *within* a process. This page covers both: what a process
actually is, how the kernel tracks it via the PCB, how context switching keeps
processes isolated and correct, and how threads extend all of this to allow
concurrency (and, on multi-core systems, real parallelism) inside a single
process.

## What Is a Process?

### Process

The process is the running instance of a program which occupies space in memory (RAM),<br> A program is a `passive entity` and process is an `active entity`. A process may passes through<br> the states as given below.

**Process memory** is divided into four sections for efficient working :

- The **Text section** is made up of the compiled program code, read in from non-volatile storage when the program is launched.
- The **Data section** is made up of the global and static variables, allocated and initialized prior to executing the main.
- The **Heap** is used for the dynamic memory allocation and is managed via calls to new, delete, malloc, free, etc.
- The **Stack** is used for local variables. Space on the stack is reserved for local variables when they are declared.

![1748664478020](/assets/images/notes/Linux_Process/1748664478020.png)

#### Process vs Program

Let us take a look at the differences between Process and Program:

| Process                                                      | Program                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| The process is basically an instance of the computer program that is being executed. | A Program is basically a collection of instructions that mainly performs a specific task when executed by the computer. |
| A process has a **shorter lifetime**.                        | A Program has a **longer lifetime**.                         |
| A Process requires resources such as memory, CPU, Input-Output devices. | A Program is stored by hard-disk and does not require any resources. |
| A process has a dynamic instance of code and data            | A Program has static code and static data.                   |
| Basically, a process is the **running instance** of the code. | On the other hand, the program is the **executable code**.   |

------------
#### [The different Process States in OS (Linux)](https://www.geeksforgeeks.org/states-of-a-process-in-operating-systems/)

Processes in the operating system can be in any of the following states:

- `NEW`- The process is being created.
- `READY`- The process is waiting to be assigned to a processor.
- `RUNNING`- Instructions are being executed.
- `WAITING`- The process is waiting for some event to occur(such as an I/O completion or reception of a signal).
- `TERMINATED`- The process has finished execution.

![different process state](/assets/images/notes/image.png)

### Types of Processes

Suppose there are two processes. One is parent process while the other is child process.<br> In a real time, there can be two scenarios:

### Orphan Process
The parent dies or gets killed before the child.
In the above scenario, the child process become**s the orphan process (as it has lost its parent).**
In Linux, the init process comes to the rescue of the orphan processes and adopts them.
This means after a child has lost its parent, the init process becomes its new parent process.

### Zombie process
**The child dies and parent does not perform wait() immediately.**
Whenever the child is terminated, the termination status of the child is available to the parent through the wait() family of calls.
So, the kernel does waits for parent to retrieve the termination status of the child before its completely wipes out the child process.
Now, In a case where parent is not able to immediately perform the wait() (in order to fetch the termination status), the terminated child process becomes zombie process.
**A zombie process is one that is waiting for its parent to fetch its termination status.**

**Although the kernel releases all the resources that the zombie process was holding before it got killed,<br> some information like its termination status, its process ID etc are still stored by the kernel<br> Once the parent performs the wait() operation, kernel clears off this information too.**

#### Daemon process

A process that needs to run for a long period of time and does not require a controlling terminal,
these type of processes are programmed in a way that they becomes a `daemon processes.`
For example, monitoring software like key-logger etc are usually programmed as daemon processes.
A daemon process has no controlling terminal.

--------------

### What is Process Scheduling?

The act of determining which process is in the **ready** state, and should be moved to the **running** state is known as **Process Scheduling**.

The prime aim of the process scheduling system is to keep the CPU busy all the time and to deliver minimum response time for all programs. For achieving this, the scheduler must apply appropriate rules for swapping processes `IN` and `OUT` of CPU.

Scheduling fell into one of the two general categories:

- **Non Pre-emptive Scheduling:** When the currently executing process gives up the CPU voluntarily. Process does not allow forcefull give up of CPU.
- **Pre-emptive Scheduling:** When the operating system decides to favour another process, pre-empting the currently executing process.

------

#### What are Scheduling Queues?

- All processes, upon entering into the system, are stored in the **Job Queue**.
- Processes in the `Ready` state are placed in the **Ready Queue**.
- Processes waiting for a device to become available are placed in **Device Queues**. There are unique device queues available for each I/O device.

A new process is initially put in the **Ready queue**. It waits in the ready queue until it is selected for execution(or dispatched). Once the process is assigned to the CPU and is executing, one of the following several events can occur:

- The process could issue an I/O request, and then be placed in the **I/O queue**.
- The process could create a new subprocess and wait for its termination.
- The process could be removed forcibly from the CPU, as a result of an interrupt, and be put back in the ready queue.

In the first two cases, the process eventually switches from the waiting state to the ready state, and is then put back in the ready queue. A process continues this cycle until it terminates, at which time it is removed from all queues and has its PCB and resources deallocated.

------

#### Types of Schedulers

There are three types of schedulers available:

1. Long Term Scheduler
2. Short Term Scheduler
3. Medium Term Scheduler

Let's discuss about all the different types of Schedulers in detail:

### Long Term Scheduler

Long term scheduler runs less frequently. Long Term Schedulers decide which program must get into the job queue. From the job queue, the Job Processor, selects processes and loads them into the memory for execution. Primary aim of the Job Scheduler is to maintain a good degree of Multiprogramming. An optimal degree of Multiprogramming means the average rate of process creation is equal to the average departure rate of processes from the execution memory.

### Short Term Scheduler

This is also known as CPU Scheduler and runs very frequently. The primary aim of this scheduler is to enhance CPU performance and increase process execution rate.

### Medium Term Scheduler

This scheduler removes the processes from memory (and from active contention for the CPU), and thus reduces the degree of multiprogramming. At some later time, the process can be reintroduced into memory and its execution can be continued where it left off. This scheme is called **swapping**. The process is swapped out, and is later swapped in, by the medium term scheduler.

Swapping may be necessary to improve the process mix, or because a change in memory requirements has overcommitted available memory, requiring memory to be freed up. This complete process is descripted in the below diagram:

![Scheduling Queues](https://static.studytonight.com/operating-system/images/process-scheduling-2-2.png)

------

#### What is Context Switch?

1. Switching the CPU to another process requires **saving** the state of the old process and **loading** the saved state for the new process. This task is known as a **Context Switch**.
2. The **context** of a process is represented in the **Process Control Block(PCB)** of a process; it includes the value of the CPU registers, the process state and memory-management information. When a context switch occurs, the Kernel saves the context of the old process in its PCB and loads the saved context of the new process scheduled to run.
3. Context switch time is **pure overhead**, because the **system does no useful work while switching**. Its speed varies from machine to machine, depending on the memory speed, the number of registers that must be copied, and the existence of special instructions(such as a single instruction to load or store all registers). Typical speeds range from 1 to 1000 microseconds.
4. Context Switching has become such a performance **bottleneck** that programmers are using new structures(threads) to avoid it whenever and wherever possible.

#### Process Control Block

There is a Process Control Block for each process, enclosing all the information about the process. It is also known as the task control block. It is a data structure, which contains the following:

- **Process State**: It can be running, waiting, etc.
- **Process ID** and the **parent process ID**.
- CPU registers and Program Counter. **Program Counter** holds the address of the next instruction to be executed for that process.
- **CPU Scheduling** information: Such as priority information and pointers to scheduling queues.
- **Memory Management information**: For example, page tables or segment tables.
- **Accounting information**: The User and kernel CPU time consumed, account numbers, limits, etc.
- **I/O Status information**: Devices allocated, open file tables, etc.

![1749038108556](/assets/images/notes/Linux_Process/1749038108556.png)


## The Process Table

### Process Table

process table in Linux (such as in nearly every other operating system) is simply a data structure in the RAM of a computer. It holds information about the processes that are currently handled by the OS.

Process table is a **kernel data structure** that describes the state of a process (along with process U Area). It contains fields that must always be available to the kernel.

This information includes general information about each process

- process id
- process owner
- process priority
- environment variables for each process
- the parent process
- pointers to the executable machine code of a process.

A very important information in the process table is the state in that each process currently is. This information is essential for the OS, because it enables the so called multiprocessing, i.e. the possibility to virtually run several processes on only one processing unit (CPU).

The information whether a process is currently ACTIVE, SLEEPING, RUNNING, etc. is used by the OS in order to handle the execution of processes.

Furthermore there is statistical information such as when was the process RUNNING the last time in order to enable the schedulr of the OS to decide which process should be running next.

So in summary the process table is the central organizational element for the OS to handle all the started processes.

### Process Control  Block

A process control block (PCB) contains information about the process, i.e. registers, quantum, priority, etc. **The process table is an array of PCB’s, that means logically contains a PCB for all of the current processes in the system**. Process Control Block is also known as Task Control Block. A Process Control Block (PCB) is a data structure.

<img src="https://media.geeksforgeeks.org/wp-content/uploads/process-table.jpg" alt="img" style="zoom:25%;" />

- **Pointer –** It is a stack pointer which is required to be saved when the process is switched from one state to another to retain the current position of the process.
- **Process state –** It stores the respective state of the process.
- **Process number –** Every process is assigned with a unique id known as process ID or PID which stores the process identifier.
- **Program counter –** It stores the counter which contains the address of the next instruction that is to be executed for the process.
- **Register –** These are the CPU registers which includes: accumulator, base, registers and general purpose registers.
- **Memory limits –** This field contains the information about memory management system used by operating system. This may include the page tables, segment tables etc.
- **Open files list –** This information includes the list of files opened for a process.

 **When the process makes a transition from one state to another, the operating system updates its information in the process’s PCB. The operating system maintains pointers to each process’s PCB in a process table so that it can access the PCB quickly.**

<img src="https://media.geeksforgeeks.org/wp-content/uploads/process-control-block.jpg" alt="img" style="zoom:25%;" />

The entry of all the PCBs of the current processes is in Process Table. The Process Table has the status of each and every process that is created in OS along with their PIDs.


## Deep Dive: Why Context Switching Works the Way It Does

### Understanding Processes in Operating Systems
Out focus is now shifting toward lower-level concepts more closely related to operating systems, such as CPU scheduling, threads, paging, and virtual memory. The challenge with these topics is that all of them are deeply tied to a concept we haven’t yet covered in detail.

![1750040691891](/assets/images/notes/Linux_process_part_1/1750040691891.png)

Today we’re going to dive into the technical details of processes. As we discussed in my previous video, a process is informally defined as a program in execution, meaning that these two concepts are not the same. A program is a passive entity, such as an executable file that we can launch to start running. When we run a program, what happens internally is that its executable file is loaded into memory. At this point, our program becomes a process.

The execution of this process, though, might require additional memory to store user input and temporary results. The operating system is responsible for allocating that memory. The memory assigned to it has a special name: the address space of the process.

![1750040838159](/assets/images/notes/Linux_process_part_1/1750040838159.png)

We’ll return to this concept later in the video, so keep it in mind. A process, however, is more than just its address space. As we know, modern operating systems use concurrency, allowing multiple processes to execute by alternating access to computer resources. Internally, alternating access to the CPU is achieved by placing processes in a queue. We'll understand how this works by the end of this video.

![1750040902137](/assets/images/notes/Linux_process_part_1/1750040902137.png)

![1750040932523](/assets/images/notes/Linux_process_part_1/1750040932523.png)

The key point now is that at any given moment, only one process can use the CPU, while all others wait their turn. Remember, the CPU has internal components like general-purpose registers, the instruction register, the address register (also known as the program counter), the stack pointer, and even flags. When a process gains access to the CPU, it uses these components to manipulate and move data. This is what running a program essentially is, as we've covered in previous episodes.
![1750041016968](/assets/images/notes/Linux_process_part_1/1750041016968.png)

But, alternating CPU access between multiple processes is not as simple as it sounds. If we simply switch processes, the process that gains access to the CPU would find itself in a CPU state belonging to the previous process. This leads to two major issues:
![1750041161219](/assets/images/notes/Linux_process_part_1/1750041161219.png)

First, this is a severe security risk since the current process could access sensitive information from the previous process. I mean, imagine if the previous process was hashing a password; part of that password could still be stored in the registers. Nothing would stop the current process from reading that information for malicious purposes. So, the **first concern here is security.**

Now, let’s assume that all processes are honest and won’t use the information from a previous process. Does that solve all the problems? Well… no. Even if the current process has no intention of using that information, it still needs to manipulate the registers to carry out its own tasks. In doing so, it alters the CPU state of the previous process. So, when the previous process regains CPU access later, the CPU state it had when it was interrupted would be lost. So, the second concern here is **correctness of execution.**
![1750085095660](/assets/images/notes/Linux_process_part_1/1750085095660.png)
![1750085103305](/assets/images/notes/Linux_process_part_1/1750085103305.png)

![1750085376877](/assets/images/notes/Linux_process_part_1/1750085376877.png)

This might be a bit difficult to follow, so let me show you an example. Let’s say we want to run two programs. When compiled to assembly, they look something like this. To be executed by the computer, the code must first be compiled into machine code. Since binary can be hard to follow, we’ll show the instructions in assembly for educational purposes. Let’s assume that both programs are launched at the same time. To run them, they are loaded into memory. And for simplicity, let’s also say the concurrency model used by the operating system allows each process to execute up to two instructions before switching CPU access to a different process.
![1750085393944](/assets/images/notes/Linux_process_part_1/1750085393944.png)

![1750085423254](/assets/images/notes/Linux_process_part_1/1750085423254.png)

To start executing the first process, the operating system sets the program counter so the CPU can begin fetching instructions for that process. The first instruction tells the CPU to load the value 12 into register 0. That’s one instruction. This process can still execute one more instruction before the operating system reallocates the CPU to the second process.The second instruction tells the CPU to load the value 20 into register 1. That completes two instructions.
![1750085497757](/assets/images/notes/Linux_process_part_1/1750085497757.png)
![1750085506263](/assets/images/notes/Linux_process_part_1/1750085506263.png)
![1750085512776](/assets/images/notes/Linux_process_part_1/1750085512776.png)
![1750085529947](/assets/images/notes/Linux_process_part_1/1750085529947.png)
![1750085539097](/assets/images/notes/Linux_process_part_1/1750085539097.png)
![1750085545773](/assets/images/notes/Linux_process_part_1/1750085545773.png)
![1750085556822](/assets/images/notes/Linux_process_part_1/1750085556822.png)
![1750085566904](/assets/images/notes/Linux_process_part_1/1750085566904.png)
![1750085584899](/assets/images/notes/Linux_process_part_1/1750085584899.png)

At this point, the operating system sets the program counter so the CPU can start executing the second process.
Now that the program counter points to the executable code of the second process, we can say the CPU is allocated to it.
![1750085604904](/assets/images/notes/Linux_process_part_1/1750085604904.png)

Note that while the second process now has control of the CPU, the data the first process was working with is still present in the registers. Again, we assume this second process is not malicious and will mind its own business. The first instruction of the second process tells the CPU to load the value 100 into register 0. That’s one instruction.
![1750085747889](/assets/images/notes/Linux_process_part_1/1750085747889.png)
The next instruction tells the CPU to load the value 35 into register 1. Now two consecutive instructions have been executed, so the operating system must reallocate the CPU back to the first process.
![1750085839613](/assets/images/notes/Linux_process_part_1/1750085839613.png)

For this, it needs to set the program counter to the correct address so the first process can continue exactly where it left off.

But… first mistake! We didn’t store that address anywhere before allocating the CPU to the second process, so now we can’t resume the first process. Here’s where things start to go wrong. Let’s assume that the operating system had saved the program counter value and is able to restore it properly. The first process regains control of the CPU and continues execution from where it was interrupted. Now the third instruction is telling the CPU to add the values currently held in registers 0 and 1, which are supposed to be 12 and 20, respectively. However, since the second process modified the registers during its execution, the CPU will now add the wrong numbers. The CPU, simply following instructions, has no way of knowing what happened, so it will continue executing the process using the incorrect data.
![1750085938158](/assets/images/notes/Linux_process_part_1/1750085938158.png)
![1750085964313](/assets/images/notes/Linux_process_part_1/1750085964313.png)
![1750085970272](/assets/images/notes/Linux_process_part_1/1750085970272.png)
![1750085999324](/assets/images/notes/Linux_process_part_1/1750085999324.png)
![1750086009593](/assets/images/notes/Linux_process_part_1/1750086009593.png)

And problems don’t stop there. In this example, the first process overwrites the value in register 0 during the addition. So, when the operating system reallocates the CPU to the second process, its CPU state will also be altered. In a similar way, the second process will continue executing, but it will end up not only adding the wrong values but also storing the wrong result. In the end, we’ve managed to mess up both processes' results. Where the first process should have produced 32, it now gives us 135. And where the second process should have produced 135, it instead gives us 170. Perhaps the worst part here is that these wrong values are not deterministic.
![1750086064621](/assets/images/notes/Linux_process_part_1/1750086064621.png)
![1750086115941](/assets/images/notes/Linux_process_part_1/1750086115941.png)
For example, if we had added a third process, or if the second program was launched even a few milliseconds later, we would’ve seen completely different wrong results. This problem becomes a nightmare when we consider that modern computers handle hundreds of processes at once. And to make matters worse, in practice, it is extremely difficult to predict the exact order in which processes will execute, as we’ll learn in the CPU scheduling video.

Ok, but then, **how do operating systems ensure security and correctness** when dealing with multiple processes? Well… the solution requires extra steps. Let’s use the same example: two processes start at the same time. We still allow each process to execute. For the first process, it loads the value 12 into register 0 and then the value 20 into register 1. After this, the CPU must be allocated to the second process. But instead of simply overwriting the address register to make the CPU jump to the executable code of the second process, the operating system first runs a special routine to capture the current state of the CPU—like taking a snapshot. The purpose of this is to copy the contents of the registers, flags, and program counter into memory so that when the process regains control of the CPU, the state it had when it was interrupted can be restored.
![1750086330276](/assets/images/notes/Linux_process_part_1/1750086330276.png)
![1750086347462](/assets/images/notes/Linux_process_part_1/1750086347462.png)

The operating system keeps a copy of the CPU state for every single process running on the computer. Right after the CPU state of the interrupted process is captured and safely stored, the CPU state of the next process is restored. In this case, since the second process hasn’t executed any instructions yet, all of its registers are set to zero, except for the program counter, which points to the next instruction the process should execute. At this moment, that would be the first instruction at memory location 1013.

![1750086371932](/assets/images/notes/Linux_process_part_1/1750086371932.png)
![1750086475863](/assets/images/notes/Linux_process_part_1/1750086475863.png)
Now, the second process starts executing, loading the value 100 into register 0 and the value 35 into register 1. After two instructions, it is interrupted to allow the CPU to be reallocated to the first process. But once again, before doing that, the CPU state of the second process is captured and safely stored. Only after storing this information does the operating system retrieve the state of the first process and copy it into the corresponding registers in the CPU.

By doing this, each time a process regains control of the CPU, it will find the registers exactly as they were when it was interrupted. This process resolves both problems. A process can no longer access the information the previous process was using, and since its own state hasn’t been altered by the other process, it can continue execution with the confidence that it is working with the correct data. This action of capturing the CPU state of a process and restoring the state of a different process so it can continue **execution is known as a context switch.** And this is how operating systems guarantee security and correctness when sharing the CPU among multiple processes.
![1750086619713](/assets/images/notes/Linux_process_part_1/1750086619713.png)
![1750086638928](/assets/images/notes/Linux_process_part_1/1750086638928.png)
![1750086663008](/assets/images/notes/Linux_process_part_1/1750086663008.png)
![1750086699902](/assets/images/notes/Linux_process_part_1/1750086699902.png)

Ok, so, now we already know that a process has an address space, a program counter, and registers, which can be informally defined as the CPU state of the process. But what else does a process have? Well, a process might also have a list of open files, as well as I/O devices allocated to it. This is the simplest way we can visualize a process. As you can see, instead of being a single entity, a process is this entire context, isolated from other processes. And that’s the best way we can describe a process with a single word: a context. This is why the kernel routine we learned about earlier is called a context switch. When we switch processes, we are replacing the entire context in which the system operates. This also explains, from a high-level perspective, why multiple processes can have the same executable instructions but still produce different results when executed. It’s not only about the instructions but also the context in which those instructions are executed.
![1750086806731](/assets/images/notes/Linux_process_part_1/1750086806731.png)
![1750086826820](/assets/images/notes/Linux_process_part_1/1750086826820.png)

Have you ever asked someone a question and received the reply, “In what context?” When someone responds this way, it’s because the same question can have different answers depending on the situation. The question that arises now is: if a process is this entire context, from low-level components like registers to higher-level things like a list of files, how can we put them in a queue? I mean, a process isn’t like an object we can simply use as an element in a data structure. So, how do we manage this? The answer is the PCB.

In this case, PCB stands for Process Control Block, a special structure the operating system uses to keep track of every single process. Since every process is unique, it requires an identifier, known as a process ID. A process also has a state, which can be any of several possible statuses. We’ll discuss these in more detail shortly.
![1750086982934](/assets/images/notes/Linux_process_part_1/1750086982934.png)
In addition, a process has a program counter, a list of general-purpose registers, an instruction register, and flags. Depending on the hardware, a process might also have a stack pointer, index registers, accumulators, and other components I won’t list here. This is what I informally refer to as the CPU state of the process.

![1750089570252](/assets/images/notes/Linux_process_part_1/1750089570252.png)

Do you remember that a context switch requires capturing the CPU state for each process? Well, this is where that data is stored.
![1750089594874](/assets/images/notes/Linux_process_part_1/1750089594874.png)
![1750089671484](/assets/images/notes/Linux_process_part_1/1750089671484.png)

Regarding memory, the operating system must also track all the memory blocks allocated to each process. Remember when we mentioned that each process has its own address space? Well, running multiple programs concurrently introduces a new security issue because, without extra precautions, any process could potentially read from or write to the address space of another process. The operating system needs to be aware of these boundaries to intercept any malicious memory access.
![1750089717382](/assets/images/notes/Linux_process_part_1/1750089717382.png)

Additionally, when a new process is created, the operating system needs to be aware of the address space of each existing process to correctly allocate an available memory region for the new process.
![1750089762587](/assets/images/notes/Linux_process_part_1/1750089762587.png)
![1750089781416](/assets/images/notes/Linux_process_part_1/1750089781416.png)
![1750089788575](/assets/images/notes/Linux_process_part_1/1750089788575.png)
Therefore, the Process Control Block should contain memory management information, at least including the memory limits of each process's address space. And here we can also add other resources allocated to the process, such as a list of I/O devices or open files.
![1750089817495](/assets/images/notes/Linux_process_part_1/1750089817495.png)

Now, the structure you’re seeing is just an example. If you want to see a real implementation, we can look at the source code of the Linux kernel under this path. The first interesting thing to note is that the structure is called a task, not a process. I’m not a kernel expert, but I think it’s called that because Linux was originally inspired by Unix, where it was said that computers ran tasks. I might be mistaken though, so feel free to correct me in the comments if I’m wrong.

**The reason it is called this way is because Linux uses the term "task" to represent the fundamental unit of execution, encompassing both threads and processes within a single data structure, effectively treating them as the same entity for scheduling purposes.** Another interesting thing I noticed when reviewing the implementation is that the Process Control Block (PCB) contains a pointer to the PCB of the parent process, as well as a list of all the processes created by the process represented by the struct. So, I guess we could add that detail to our example. Again, I want to emphasize that the Process Control Block is not the process itself, but rather a representation of the process. It serves as a repository for all the data needed to start or resume a process, along with some accounting information. And this representation is what is actually placed in a queue.
![1750089997659](/assets/images/notes/Linux_process_part_1/1750089997659.png)
![1750090019169](/assets/images/notes/Linux_process_part_1/1750090019169.png)

And at this point, we should be ready to dive into CPU scheduling, a topic for a future episode. Finally, keep in mind that everything we’ve covered in this video is valid for single processor systems and multicore systems. The only difference is that multicore systems can operate with multiple contexts at the same time due to each core having its own execution pipeline.
![1750090102544](/assets/images/notes/Linux_process_part_1/1750090102544.png)
![1750090115178](/assets/images/notes/Linux_process_part_1/1750090115178.png)


## Threads vs Processes

### Thread vs Process

### **Comparison Chart**

|                                                 **Process**                                                 |                                   **Thread**                                   |
| :---------------------------------------------------------------------------------------------------------------: | :-----------------------------------------------------------------------------------: |
|                              Process is also called a **heavy-weight** process.                              |                  Thread is also called **light-weight** process                  |
|                           Operating system interface is required for process switching.                           |                Operating system is not required for thread switching.                |
|                             Each process operates independently of the other process.                             |     One thread can read, write or even completely clean another thread’s stack.     |
|      In multiple processing, each process executes the same code but has its own memory and file resources.      |         All threads can share the same set of open files and child processes.         |
|                                         More Time required for creation.                                         |                           Less Time required for creation.                           |
|                                       System calls involved in the process.                                       |                               No system calls involved.                               |
| If one server process is blocked then other server processes cannot execute until the first process is unblocked. | If one thread is blocked and waiting then the second thread in the same task can run. |
|                                               Uses more resources.                                               |                                 Uses fewer resources.                                 |

![1749192297755](/assets/images/notes/Linux_Thread_vs_Process/1749192297755.png)
![1749192417733](/assets/images/notes/Linux_Thread_vs_Process/1749192417733.png)

***Thread stacks can be located anywhere in the `process's virtual address space.`
They are managed by the kernel via mmap, and there's no requirement for them to be within or between the main thread's stack and heap.***

> **Is it possible that process stack can enter in the boundaries of thread stack ?**

❌ No, under normal and correct operation, a process's main stack cannot intrude into another thread’s stack. Here's why — and how stack boundaries are strictly managed.

> **⚠️ Can Stack Overflow Happen?**

 Yes, but it doesn't cross into other threads' stacks unless you're violating memory protections. Two ways this can go wrong:

 1. **Manual Stack Mismanagement (e.g. unsafe pthread_attr_setstack)**
    If you set overlapping or too-small stacks manually, undefined behavior can occur.

 2. **Massive Recursion / Buffer Overflow**
    If you blow past the stack limit, you might corrupt adjacent memory or get a segmentation fault,
    but the OS will usually catch it first via guard pages.


## What Is a Thread, and Why Do We Need One?

### Understanding Threads and Concurrency

![1750004805879](/assets/images/notes/Linux_thread-part-1/1750004805879.png)

![1750004818167](/assets/images/notes/Linux_thread-part-1/1750004818167.png)

But multitasking is only the well-known purpose of concurrency and CPU scheduling. There's another, less obvious purpose: maximizing the use of computer resources. We'll dive deeper into this in my video about CPU scheduling. For now, what we need to know is that while a process is often defined as a program in execution, it doesn't necessarily mean the process is always using the CPU. Even if the CPU is allocated to that process, for example, a process might be waiting for an I/O resource. In that scenario, allocating the CPU to that process would be inefficient because it can't execute instructions until the requested I/O resource is ready. So instead, it makes sense to allocate the CPU to a different process—one that's ready to execute instructions.

![1750004889100](/assets/images/notes/Linux_thread-part-1/1750004889100.png)

![1750004918441](/assets/images/notes/Linux_thread-part-1/1750004918441.png)

![1750004973079](/assets/images/notes/Linux_thread-part-1/1750004973079.png)

This is the second reason why concurrency is so important: it's not just about running multiple programs at once; it's also about filling those little gaps when a process can't use the CPU by assigning it to another process that can. Up **until now, in this series, we've treated processes as the fundamental unit of execution, so we've assumed there's nothing more basic than a process for scheduling purposes. So we can't employ concurrency for executable entities within the same process.**

If this sounds confusing, what I'm trying to say is that tasks within the same program, like two functions, for example, cannot run asynchronously under the traditional process approach. This makes sense if you've watched my video on processes. Remember, each process has one program counter that points to the instruction where it is interrupted when the CPU is allocated to another process. But we cannot alternate the CPU between two functions that are part of the same process because, with a single program counter, we can't keep track of where both are interrupted. One of the functions must wait for the other to finish executing.

![1750005042962](/assets/images/notes/Linux_thread-part-1/1750005042962.png)

![1750005077003](/assets/images/notes/Linux_thread-part-1/1750005077003.png)

![1750005083978](/assets/images/notes/Linux_thread-part-1/1750005083978.png)

**But why would we need concurrency inside a process in the first place? I mean, if we wanted two pieces of code to run concurrently and cooperate, couldn't we just put that code in separate processes and use interprocess communication to coordinate their actions? Well, yes, but using IPC isn't always intuitive. Sometimes different pieces of code that contribute to a main goal are so correlated that putting them in different processes just feels wrong. There are many situations where concurrency within a process is incredibly useful.**

Think about a server that receives requests on port 3000, where its sole purpose is to return a profile image associated with each client. When a client sends a request, the server accepts it, processes it by searching and reading the image from disk, and once the image is loaded into memory, responds by sending the image back to the client. Because images are stored as files on disk, a very costly I/O operation is required to find the image and load it into memory. As a result, processing a request can take significantly longer than simply accepting or responding to it. These steps are executed sequentially and obviously take only a few milliseconds, so for a single client, it works just fine. But with multiple clients, complications arise.

If five requests arrive at approximately the same time, the server will accept the first one quickly but then spend significant time processing it. The second request will only be handled once the first is completed, and so on. This creates a bottleneck, with later requests waiting a long time to be attended to. Now, with five requests, the bottleneck is not that much of a problem, but imagine a thousand requests, and here we can see the real problem. Notice the gray gaps in the timeline; these represent periods where the CPU sits idle, doing nothing. This is wasted computational time—time that could have been used to accept and begin processing other clients' requests. When one task is unable to execute because another task is running, despite the two being independent, we call it a blocking effect.

![1750005157525](/assets/images/notes/Linux_thread-part-1/1750005157525.png)

![1750005147983](/assets/images/notes/Linux_thread-part-1/1750005147983.png)

![1750005227199](/assets/images/notes/Linux_thread-part-1/1750005227199.png)

For a long time, the solution to this problem was as follows: there's a main process that listens for requests, which we'll call the Listener process. Every time a client sends a request, instead of handling it directly, the Listener process creates a whole new process to attend to that specific client. This way, if another client sends a request, it won't wait until the previous request is completely handled because the process that listens and accepts the requests runs concurrently with the one serving the previous client. In other words, this method allows taking advantage of concurrency to use the CPU as much as possible.

![1750005288727](/assets/images/notes/Linux_thread-part-1/1750005288727.png)

![1750005296951](/assets/images/notes/Linux_thread-part-1/1750005296951.png)

While clever, this approach isn't perfect. Remember, each process is like a self-contained context with its own properties, including an address space. Creating an entire process for every client might work for a few hundred clients, but when scaling to thousands, it becomes inefficient in terms of memory usage. Additionally, spawning a new process is not a trivial operation; it consumes processing time. So even though the goal is to utilize idle CPU gaps, a portion of that time is actually spent creating the new process itself. Moreover, if the server needs to handle some kind of global state, all child processes must be synchronized to keep track of it. This requires some form of IPC, complicating the implementation.

![1750005353927](/assets/images/notes/Linux_thread-part-1/1750005353927.png)

![1750005390903](/assets/images/notes/Linux_thread-part-1/1750005390903.png)

**Although processes are one of the most important concepts in computer science, they aren't perfect.**  the concept of a process, which is very useful. We can't just discard it, but we can modify it slightly to allow concurrency within the executable code of a single process. Remember, in general terms, a process comprises an ID, a program counter, a register set, an address space, and other resources such as open files and I/O devices. As I explained earlier, the main limitation is that a single program counter doesn't allow the process control block—the structure the OS uses to represent processes—to keep track of more than one task at a time.

The solution is to stop associating the program counter directly with the process and instead assign a program counter to each inner executable entity that we want to run concurrently within the process. These inner entities are what we call threads. By no longer limiting each process to a single program counter, we can now create a new thread whenever we need code within the same process to run concurrently. This solves the blocking problem without creating new processes. If one of the threads needs an I/O resource, such as loading the contents of a file into an array, the other threads can continue executing without being blocked.

![1750005525488](/assets/images/notes/Linux_thread-part-1/1750005525488.png)

![1750005558296](/assets/images/notes/Linux_thread-part-1/1750005558296.png)

![1750005647530](/assets/images/notes/Linux_thread-part-1/1750005647530.png)

While threads share the entire address space base of the process they belong to, they cannot share a CPU state. Because threads are going to be interrupted to allocate the CPU to other threads, we need to capture the state of each thread so that when the CPU is reallocated, its state can be restored. Therefore, if each thread has its own program counter, it also must have its own register set, flags, accumulators, etc.—essentially its own CPU state. And note that this includes a separate stack pointer. Remember, the stack is a fast and efficient way to organize and access local variables in memory. If two functions in the same process run concurrently and share the same stack, they could easily overwrite each other's data. Hence, each thread must have its own stack.
![1750005961813](/assets/images/notes/Linux_thread-part-1/1750005961813.png)

![1750005979628](/assets/images/notes/Linux_thread-part-1/1750005979628.png)

**Something that I want to make sure is very clear is that the fact that each thread has its own stack doesn't mean that they cannot read from or write to each other's. This makes perfect sense if we consider the address space as a property of the process, not the threads. Since the stacks are located within that shared address space, nothing prevents a thread from accessing another thread's stack.** Whether you should do it or not is a completely different discussion, but generally, unless you really know what you're doing, it's best to avoid accessing other threads' stacks directly.

If threads within the same process need to communicate by sharing memory, the heap is more appropriate for that. Since by its nature, the data stored in the heap is not organized in a predictable way anyway. But even with the heap, caution is still required. If one thread is reading from a block of memory while another is writing to it, the results can be disastrous. Synchronizing concurrent tasks is so critical that mechanisms for doing so are implemented directly in hardware. I'll cover this in a future video.

Completely discarding the concept of a process, address spaces of different processes are still isolated. So while sibling processes share memory by default, if threads from different processes want to share data, interprocess communication is needed. Replacing the traditional multi-process approach with the multi-threaded approach requires reimplementing the process control block. The most obvious path here would be to use a second structure to represent threads. While implementation details can vary by platform, the most common approach is the one used in the Linux kernel, where both processes and threads are represented by the same structure called task. If you ask me, I'd say this is a perfect example of good naming in programming because it abstracts away the distinction between processes and threads.

So instead of describing concurrency as alternating CPU access between processes, threads, or both, we can simply describe it as alternating tasks. This implementation is more intuitive if we want to use a single scheduler for threads and processes. Regardless of the approach, doing this requires that every process has at least one thread called the main thread. Whenever a new process is created, the operating system automatically creates this thread. So even if a process doesn't rely on multi-threading, the operating system can allocate the CPU to it by scheduling its main thread. Whether a process starts with a fixed number of additional threads or can spawn threads dynamically depends on the operating system's implementation. Most mainstream operating systems prefer the dynamic approach, where each process starts as a single-threaded process and additional threads are created at runtime if needed. This, of course, increases the complexity of the operating system's internal implementation, as it must provide system calls for dynamic thread creation. But it's the preferred method because, in many cases, it's impossible to know at compile time how many threads will be needed at runtime.

![1750006028549](/assets/images/notes/Linux_thread-part-1/1750006028549.png)

![1750006047825](/assets/images/notes/Linux_thread-part-1/1750006047825.png)

![1750006053091](/assets/images/notes/Linux_thread-part-1/1750006053091.png)

![1750006059564](/assets/images/notes/Linux_thread-part-1/1750006059564.png)

![1750006066674](/assets/images/notes/Linux_thread-part-1/1750006066674.png)

![1750006088569](/assets/images/notes/Linux_thread-part-1/1750006088569.png)

![1750006197616](/assets/images/notes/Linux_thread-part-1/1750006197616.png)

Okay, that's a lot of information! Before we discuss how all of this applies to our server example, there's one last thing we need to know. In many implementations, because the main thread identifies the process it belongs to, if other threads are spawned and the main thread terminates its execution, the execution of all the other threads will immediately terminate. There are multiple ways to handle this problem, but that's a topic for another video.

Back to our server example: with the multi-threading approach, we can spawn a new thread for each incoming client, achieving a similar effect to the multi-process approach but with far less memory consumption. Additionally, there can be a performance improvement when using threads because the system calls used to create threads are generally completed much faster than those used to create new processes. The reasons for this should be obvious at this point.

And that being said, we should now define what a thread is. But first, we'll define what a thread is not. A thread is not a function, and any code that a thread executes is in the text section of the memory layout of the process. **The thread doesn't contain code; it points to the code through its program counter. This means multiple threads can point to the exact same executable code.** This is precisely what happens in our server example since all the requests need to be handled in the same way. All the handler threads don't contain a function but rather point to the same function in memory.

**If you're wondering if threads accessing this memory area at the same time is dangerous, it's not, because executable code and constants reside in the text and data sections—two regions that are never written to. All the information pertinent to a specific thread generated at runtime resides either on the stack or the heap. So threads accessing the same executable code concurrently is perfectly safe.**

![1750007008206](/assets/images/notes/Linux_thread-part-1/1750007008206.png)

![1750007279524](/assets/images/notes/Linux_thread-part-1/1750007279524.png)

![1750007286480](/assets/images/notes/Linux_thread-part-1/1750007286480.png)

![1750007318244](/assets/images/notes/Linux_thread-part-1/1750007318244.png)

![1750007328505](/assets/images/notes/Linux_thread-part-1/1750007328505.png)

Perhaps this last part helps you understand why, in low-level languages like C, spawning a thread requires us to pass a pointer to a function as a parameter. This might seem confusing, but what we're actually doing is passing the memory address where the code that the thread will execute begins. So please don't misinterpret my animations: threads are not asynchronous functions.

![1750007366635](/assets/images/notes/Linux_thread-part-1/1750007366635.png)
![1750007380997](/assets/images/notes/Linux_thread-part-1/1750007380997.png)
![1750007411543](/assets/images/notes/Linux_thread-part-1/1750007411543.png)
![1750007091618](/assets/images/notes/Linux_thread-part-1/1750007091618.png)
![1750007162624](/assets/images/notes/Linux_thread-part-1/1750007162624.png)
![1750007175607](/assets/images/notes/Linux_thread-part-1/1750007175607.png)

There are two ways we can define threads. From the operating system's point of view, threads are what processes were in our previous episode: the most basic unit of execution. From a developer's perspective, threads are a mechanism to tell the operating system that certain pieces of code inside our program can be executed concurrently. Another interesting definition that I've heard is that threads can be seen as lightweight processes—easier and faster to create.

And remember, everything I've said in this video applies even to systems that dispose of a single-core processor. Let's wrap things up for now. In the next episode, we'll discuss the additional purpose of threads in systems where the processor has multiple cores: the fundamentals of parallelism.

### Summary

**Understanding Threads and Concurrency**

🧠 Core Concepts:
Concurrency allows multiple processes to appear to run simultaneously on a single CPU via time-sharing.

CPU scheduling ensures efficient use of CPU time by switching between processes when one is waiting (e.g., for I/O).

Concurrency is not just multitasking—it's also about filling CPU idle gaps effectively.

📦 Processes vs. Threads:
A process is a self-contained unit with its own memory and program counter.

Traditional concurrency only alternates between processes, not functions inside a process.

This leads to limitations like blocking, where tasks inside one process can’t execute concurrently.

⚙️ Why Threads?
Threads allow concurrent execution within the same process.

Each thread has its own:

Program counter

Registers

Stack

All threads in a process share:

Address space

Heap

Code (text section)

🌐 Server Example:
A server processing requests sequentially creates bottlenecks due to I/O wait time.

Early solution: spawn a new process per client → expensive (memory and time).

Better solution: spawn a thread per client → lightweight, faster creation, shared memory.

🔄 Synchronization:
Threads need synchronization when accessing shared memory to prevent data corruption.

Especially critical when multiple threads access the heap.

🧩 Implementation Notes:
In Linux, both threads and processes are represented as tasks, enabling unified scheduling.

All processes start with a main thread, and more threads can be created dynamically.

Termination of the main thread often terminates all other threads unless handled properly.

✅ Thread Definitions:
OS view: Basic unit of execution.

Developer view: Mechanism to enable concurrent code execution within a process.

Threads are often called lightweight processes.

🔐 Safety & Execution:
Threads don’t contain code—they point to code.

Concurrent execution of the same code is safe as text/data sections are read-only.


## Threads on Multi-Core Systems: Concurrency vs. Parallelism

### Threads and Multi-Core Processors

In the last episode, we learned about threads, which are very useful when writing programs for two key reasons. If one task in the same program takes a long time to complete and another takes only a few milliseconds, concurrency ensures the shorter task doesn't have to wait a long time to start running. If a task is waiting for an I/O resource, such as a file read or a network response, it can't use the CPU during that time. Instead of letting the CPU sit idle, the operating system can allocate that unused time to another task that's ready to run.

So, threads are simply a way to tell the operating system that multiple tasks within the same process can run concurrently. They enable applications to perform multiple tasks at the same time. For example, in an email client app, we need to display the user interface on the screen while listening for the user keystrokes, while uploading an attachment like a photo from your file system, while performing grammar checking, and while monitoring for incoming emails. In order to provide a good user experience, none of these tasks should wait for each other to complete.

![1750012342805](/assets/images/notes/Linux_thread-part-2/1750012342805.png)
![1750012380478](/assets/images/notes/Linux_thread-part-2/1750012380478.png)

Implementing concurrency has some challenges, though. If the number of concurrent tasks increases too much, at some point, the system won't feel smooth anymore. Even if the CPU alternates between tasks extremely quickly, there comes a point where there are so many tasks that it takes too long for each one to regain access to the CPU. To address this, there are three possible solutions. The most obvious would be to simply make the CPU faster. If the CPU can handle more work in the same amount of time, tasks can regain access to it more quickly. This solution, though, isn't perfect because if we keep increasing the number of tasks, we will eventually end up back where we started: too many tasks causing delays. Plus, making CPUs faster has become increasingly difficult over the last decade.

The second solution would be to schedule CPU access in a more clever way. This is a more complicated solution that deserves its own video. The third solution, like the first one, is more of a brute force approach: if a single processor can't handle too many tasks, no matter how fast it is, just add more processors. This can be achieved in several ways: by adding more CPU sockets to the same motherboard, allowing for multiple physical CPUs, or by including multiple processing units within a single package or chip, known as multi-core processors. And, though less common, by combining multiple multi-core chips in the same motherboard.

![1750012947748](/assets/images/notes/Linux_thread-part-2/1750012947748.png)

![1750013009439](/assets/images/notes/Linux_thread-part-2/1750013009439.png)

![1750013052506](/assets/images/notes/Linux_thread-part-2/1750013052506.png)
![1750013068630](/assets/images/notes/Linux_thread-part-2/1750013068630.png)

The terminology around processors can get a bit ambiguous. The term CPU is often used to refer to the entire package or chip; however, each core inside the package works as an independent processing unit, essentially a CPU that shares some components, like the cache, with other cores. In any case, each core appears as a separate processor to the operating system. So, if you heard the term "cores" a lot in this video, you know what I mean.

An application with eight threads on a system with a single computing core means that concurrency merely means that the execution of the threads will be interleaved over time because the processing core is capable of executing only one thread at a time.

![1750013211283](/assets/images/notes/Linux_thread-part-2/1750013211283.png)

But on a system with multiple cores, concurrency takes on a new meaning. Here, some threads can truly run at the same time because the system can assign each thread to a separate core. In other words, with multiple cores, we're no longer just dealing with concurrency; we're dealing with parallelism.

![1750013259486](/assets/images/notes/Linux_thread-part-2/1750013259486.png)

![1750013271109](/assets/images/notes/Linux_thread-part-2/1750013271109.png)

Notice the distinction between concurrency and parallelism in this discussion. A concurrent system supports more than one task by allowing all the tasks to make some progress. In contrast, a parallel system can perform more than one task truly simultaneously. Thus, it is possible to have concurrency without parallelism. One of the main advantages in parallel systems is that the smoothness of multitasking becomes less reliant on the illusion created by fast interleaving. Suddenly, it makes even more sense why virtually all modern operating systems consider threads, rather than processes, as the basic unit of execution. With multi-core processors now the standard, threads within the same process can take full advantage of parallelism.
![1750013939065](/assets/images/notes/Linux_thread-part-2/1750013939065.png)
![1750013947334](/assets/images/notes/Linux_thread-part-2/1750013947334.png)
![1750013812816](/assets/images/notes/Linux_thread-part-2/1750013812816.png)

For us as programmers, this means that if we want to run tasks in parallel, all we need to do is declare those tasks as concurrent using threads. The operating system will handle the rest, interleaving the CPU between threads if no additional core is available or assigning one core to each thread if the system runs on a multi-core processor. This makes our programs more portable since we don't have to compile for a specific number of cores. Just always consider two important things: the number of cores is fixed, so creating a thousand threads doesn't mean that a thousand tasks will run in parallel. Instead, if the system has n cores, up to n threads can execute in parallel at any given time. And pay attention to this: up to n, because threads compete for resources. Even if we create the exact number of threads as the number of cores available, threads from other processes also need CPU time. Since one of the main goals of the operating system is to ensure fair distribution of CPU resources across all threads, it limits how many of our own threads can run in parallel.

![1750013887429](/assets/images/notes/Linux_thread-part-2/1750013887429.png)
![1750013899065](/assets/images/notes/Linux_thread-part-2/1750013899065.png)

With that being said, let's discuss another reason why we might need parallelism in our programs: performance. This one is pretty obvious. If we can truly run more than one task at the same time, we can significantly reduce the total time it would take to complete those tasks compared to running them sequentially on a single-core system.

In general, there are two types of parallelism: data parallelism and task parallelism. Data parallelism focuses on distributing subsets of the same data across multiple computing cores and performing the same operation on each core. Task parallelism involves distributing not data, but tasks or threads across multiple computing cores. In other words, each thread is performing a unique operation. But here we have multiple scenarios: different threads may be operating on the same data, or they may be operating on different data.

![1750014213025](/assets/images/notes/Linux_thread-part-2/1750014213025.png)

Let's say we have a large data set of numbers stored in an array, and our task is to find all the prime numbers in that array. This problem requires us to iterate over the entire array, checking whether each number is prime. The key here is to understand that the result of checking whether a number is prime doesn't depend on the results for any other numbers in the array. If we want to check whether the last number in the array is prime, we don't need to wait for the earlier numbers to be checked. Each number can be processed independently.

Now, if we have a four-core processor, we can split the data into four equal parts and assign each part to a different core. This is an example of data parallelism because all cores are performing the same operation on distributed subsets of the data. However, keep in mind that splitting the work across four cores doesn't necessarily mean we'll get four times the performance. For example, I tested the same example on my computer over 10 million times, and here's the average time it took to compute each subset. How much performance gain we can achieve from parallel operations is beyond the scope of this video.

![1750014257228](/assets/images/notes/Linux_thread-part-2/1750014257228.png)

![1750014281353](/assets/images/notes/Linux_thread-part-2/1750014281353.png)

But I'd say we're given the same data set, but this time we're tasked with finding the lowest value in the array, finding the highest value, calculating the arithmetic mean of all the elements, and checking if the array contains the number 101. In this case, it doesn't make sense to split the data into subsets because each of these operations requires access to the entire data set to compute their results. But here's what we can do: assign each operation to a different core. This is an example of task parallelism—different threads working with the same data set but performing different operations. Again, using four cores doesn't mean the process will be four times faster, but it's still a significant improvement. On a single-core system, we'd have to perform these four operations one after the other. With four cores, we can execute them simultaneously, reducing the total time.

![1750014365989](/assets/images/notes/Linux_thread-part-2/1750014365989.png)

And that's about it for now. There's more content about threads coming soon.

### Summary

**Threads and Multi-Core Processors**
Concurrency allows multiple tasks to make progress by sharing CPU time, especially useful when one task is idle (e.g., waiting for I/O).

Threads enable multiple tasks within the same program to run independently, improving responsiveness (e.g., in an email app).

On a single-core system, threads are interleaved—only one runs at a time.

On multi-core systems, threads can run truly in parallel, each on a separate core—this is parallelism.

The OS schedules threads either by interleaving or distributing them across available cores.

Three solutions to handle too many concurrent tasks:

Make the CPU faster (limited scalability).

Improve task scheduling (complex).

Add more processors (multi-core systems).

Terminology: Each core in a CPU acts like a separate processor, even if they share some components.

Concurrency ≠ Parallelism:

Concurrency: tasks make progress.

Parallelism: tasks truly run at the same time.

Threads are preferred over processes as the basic execution unit in modern operating systems.

Creating more threads than cores doesn't mean all run in parallel—only up to n threads can run simultaneously on n cores.

Two types of parallelism:

Data Parallelism: same operation on different data chunks (e.g., checking primes in array parts).

Task Parallelism: different operations on same/full dataset (e.g., finding min, max, mean).

Parallelism boosts performance but doesn’t scale linearly due to overhead and resource contention.

Examples illustrate how dividing work across cores can save time—but gains depend on task and system setup.

