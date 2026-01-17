# Socket Programming :

- What is a socket? What is it used for?
- How are IP addresses resolved? What is the difference between an IP4 and IP6 address? How does that affect the socket calls?
- How do you determine what port to connect to? What are some of the common ones? What ports wouldn't you use?
- What is the difference between a TCP socket and a UDP socket? Why would you use one or the other?
- What list of calls are used to establish a TCP and/or UDP socket connection? What information is needed?
- How are the socket calls different if you are establishing a server side socket?
- How do you test whether a socket is ready to be read or written?
- If a socket closes unexpectedly, how would you know and how would you handle it?
- What happens to the connection, thread, process if a socket is not closed out properly?
- How long does a socket connection stay open when idle? How would you change that behavior?
- How would you set a socket to non-blocking mode? In what situations would you do that?
- How would you handle writing a multi-process and/or multi-threaded socket application? What considerations are there? Is there more than one way to handle it?

# Memory Managment :

- What is the difference between logical, physical and virtual memory in Linux
- When do you use ioremap and when do you use mmap
- You want to access a physical memory(you know the address, assume registers or some device address) in user-space/kernel-driver, how would you do that
- What is high/low memory in Linux?
- You have a PCIe device connected how would you access the PCIe device memory in application code?
- What is a memory map, describe memory map of some system which you are aware of
- What problem do you face when DMA is operating on a cacheable memory and how would you solve the issue
- Page Table and its handling
- L1/L2 cache in Linux
- **Knowledge questions:**

  - (+) Describe the memory hierarchy. (I ask this in all operations interviews - no exceptions.)
  - (+) Understand basic SMP architecture. Be able to diagram it.
  - (+) How is memory addressed?
  - [How are programs and libraries loaded into memory?](./Linux_IPC_socket.md#Section-1)
  - [In microcontrollers, the .text section executes directly from Flash while .data and .bss are loaded into SRAM, but in Linux systems, the .text section is also loaded into RAM as part of the virtual memory mapping. Why?](./Linux_IPC_socket.md#section-2)
  - [What is segmentation?](./Linux_IPC_socket.md#section-3)
  - How does the kernel handle its own memory? What’s the difference between kernel memory and user memory? What is shared memory?
  - What is paging? What hardware mechanisms are involved? What software mechanisms?
  - (+) [What’s the difference between physical and virtual memory?](./Linux_IPC_socket.md#section-4)
  - [What is swapping? How does it relate to paging?](./Linux_IPC_socket.md#section-5)
  - [What is thrashing?](./Linux_IPC_socket.md#section-6)
  - (+) [What are page faults?](./Linux_IPC_socket.md#section-7)
  - [Be prepared to handle questions about the malloc family of functions. What do they do, how do they work, what are common pitfalls?](./Linux_IPC_socket.md#section-8)
  - (+) [Be prepared to handle questions about the fork, exec, clone functions (and their relatives) and how they relate to process memory.](./Linux_IPC_socket.md#section-9)
  - Be prepared to answer questions about POSIX shared memory: mmap, shm_open, fstat, and similar functions.
  - (+) [What is DMA? How does it work?](./Linux_IPC_socket.md#section-10)
  - (+) [What is a cache? What is a buffer? Compare and contrast them. What are common examples of code or subsystems where you might encounter them?](./Linux_IPC_socket.md#section-11)

  **Practical questions:**

  - What tools can you use to find the overall memory utilization and performance of the memory subsystem?
  - How do you find out information about an individual process’s memory utilization? (e.g. resident set size, virtual set size, paging, etc.)
  - Where would you expect to find memory related errors logged?
  - (+) What debugging utilities are available for diagnosing and evaluating memory issues in applications? When would you use each one? (e.g. strace, gdb, valgrind, core, etc.)
  - What type of information about process memory can be found in /proc?
  - How can the memory subsystem setting parameters be tuned? What are some common changes?
  - Describe the impacts of spatial and temporal locality on application performance.

  **Special topics** (unlikely you’d encounter these unless you’re interviewing in very specific environments):

  - memtest and similar hardware diagnostics. When should you suspect something is wrong?
  - What constitutes effective alerting about memory utilization?
  - What is the OOM killer? Why is it important?
  - How does containerization relate to memory management?
  - How does NUMA differ from SMP? How are they similar? Where and when do you expect to find each?
  - What is COW and why was it incredibly important in regards to Linux security recently (c. 2016)?
  - What is DMA? RDMA? Where would you expect to find them and why?
  - Compare and contrast LRU, LFU, MFU, FIFO, etc. paging algorithms.
  - How do type 1 hypervisors affect clients’ memory subsystem? Where and when does this become visible?

