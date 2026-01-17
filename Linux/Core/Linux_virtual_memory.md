### Section-4

| Feature                      | **Physical Memory**             | **Virtual Memory**                                |
| ---------------------------- | ------------------------------------- | ------------------------------------------------------- |
| **Definition**         | Actual hardware RAM                   | Abstraction of memory provided by OS + hardware         |
| **Location**           | RAM (primary memory)                  | Lies in RAM and/or disk (via swap/paging)               |
| **Size Limit**         | Limited by installed RAM (e.g., 8 GB) | Limited by CPU architecture (e.g., 48-bit VA → 256 TB) |
| **Managed by**         | Hardware (memory controller)          | OS + MMU (Memory Management Unit)                       |
| **Accessed by**        | OS kernel directly                    | User processes and kernel use virtual addresses         |
| **Address Type**       | Physical addresses (PA)               | Virtual addresses (VA)                                  |
| **Security/Isolation** | No isolation across processes         | Isolates processes (each has its own address space)     |
| **Swapping Possible?** | ❌ No                                 | ✅ Yes (RAM ↔ disk)                                    |

Virtual memory helps us to resolve 3 problems.
1) Not Enough memory
2) Memory Fragmentation
3) Security

![1749364366483](image/Linux_IPC_socket/1749364366483.png)

![1749364615901](image/Linux_IPC_socket/1749364615901.png)

![1749364799372](image/Linux_IPC_socket/1749364799372.png)

![1749396328022](image/Linux_IPC_socket/1749396328022.png)

![alt text](image/image-2.png)

OS uses special data structure to map virtual memory pages* with physical memory frame*, known as `Page table`.
Every Page table entry includes physical address of the page in RAM and some additional information i.e protection, status etc.
Number of entries in a single page table can be huge.
If you have 16GB of RAM and each page size is 4KB than 4 million pages can accomadate in `16GB RAM memory.`

Since each process has their own page table. If you consider one Page table entry requires `4 Bytes` of memry than each Page table requires `16MBytes` of memory. If you have 50 process then we need `800MB` just to keep Page table. which is not acceptable.

![alt text](image/image-4.png)

![1749313255324](image/Linux_IPC_socket/1749313255324.png)

![1749313539369](image/Linux_IPC_socket/1749313539369.png)

![1749313508331](image/Linux_IPC_socket/1749313508331.png)

![1749313470308](image/Linux_IPC_socket/1749313470308.png)

If CPU need to access RAM every time to translate the virtual to physical memory then too many read and write operations are not acceptable. Memory access is slow process as compare to processor speed.
Here come one hardware to rescue name MMU(memory management Unit). MMU manages all memory request coming form processor.
Its primary jobs to translate virtual memory address into physical memory addresses.
It also help to restrict other process to access other's private or reserve memory.

![1749313809394](image/Linux_IPC_socket/1749313809394.png)

![alt text](image/image-3.png)

![1749314138274](image/Linux_IPC_socket/1749314138274.png)

![1749314415297](image/Linux_IPC_socket/1749314415297.png)

![1749314473340](image/Linux_IPC_socket/1749314473340.png)

![1749314498915](image/Linux_IPC_socket/1749314498915.png)

**TLB entrie are very simialry to page table entry except ASID. ASID help to store multiple process page table data.
if ASID will not be there than every time context switch happen, TLB need to flush and repopulate on every new process came.
than there will no use of MMU itself.**

![1749314558327](image/Linux_IPC_socket/1749314558327.png)

When any instruction require memory access, processor send virtual address to MMU. Which takes care everything.
virtual address split into two parts.
**Page number :** sequencial page number in virtual memory
**page offset :** Bytes within the page.

![1749314898684](image/Linux_IPC_socket/1749314898684.png)

![1749315026718](image/Linux_IPC_socket/1749315026718.png)

![1749315091979](image/Linux_IPC_socket/1749315091979.png)

![1749315174668](image/Linux_IPC_socket/1749315174668.png)

if Page number does not found in TLB than MMU send a request to look page in full Page table. It is slow process. Once found than entry is checked for validaty.

![1749315405863](image/Linux_IPC_socket/1749315405863.png)

![1749315454726](image/Linux_IPC_socket/1749315454726.png)

if `frame number` is invalid than MMU genertate page fault. which need to send OS and OS takes care for that.<br> OS check that frame should not be form restricted area otherwise segmentation fault.

![1749315545014](image/Linux_IPC_socket/1749315545014.png)

![1749315629724](image/Linux_IPC_socket/1749315629724.png)

![1749315732924](image/Linux_IPC_socket/1749315732924.png)


### Section-5

🔄 What is Swapping?
Swapping is a memory management technique where the operating system moves entire processes (or parts of them) from RAM to disk (and back) to free up physical memory (RAM) for other tasks.

It’s part of how virtual memory works, allowing the system to run more processes than the available physical memory can hold.

📦 What is Paging?
Paging is a low-level mechanism that divides memory into fixed-size chunks:

Pages (typically 4 KB) in virtual memory

Frames (same size) in physical memory (RAM)

The OS maintains a page table to map virtual pages to physical frames.

| Concept            | Description                                |
| ------------------ | ------------------------------------------ |
| **Paging**   | Divides memory into pages and maps them.   |
| **Swapping** | Moves pages**between RAM and disk**. |

So:

Paging sets up the framework for using virtual memory via pages.

Swapping uses that framework to move pages in and out of RAM when needed.

🔁 Flow: What Happens in a Page Swap?
Process accesses a virtual address.

MMU checks page table → page is not in RAM → page fault occurs.

OS:

Finds a free page in RAM (or evicts an existing one).

Loads the needed page from swap space (disk) into RAM.

Updates page table with new mapping.

This is called demand paging

### Section-6

🔥 What is Thrashing?
Thrashing occurs when a system spends more time swapping pages in and out of memory than actually executing processes.
It’s a performance disaster caused by excessive paging due to memory overcommitment.
CPU is busy handling page faults instead of running application code.

📉 In Simple Terms:
Too many processes + too little RAM = Constant page faults
→ System becomes extremely slow or unresponsive.

🔁 Thrashing Behavior Pattern
Process A needs Page X → not in RAM → page fault

OS swaps out Page Y to make room

Process B now needs Page Y → not in RAM → another page fault

Repeat...

System gets stuck in this vicious cycle:
⚠️ Swap → Page fault → Swap → Page fault → …

### Section-7

💥 What is a Page Fault?
A page fault is an event that occurs when a process tries to access a part of its virtual memory that is not currently mapped to a physical page in RAM.

📌 Simple Definition:
A page fault happens when a program uses a memory address that is not in RAM, and the OS has to fetch the required page from disk or elsewhere to continue execution.

| Type                         | Description                                                                                               |
| ---------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Minor Page Fault**   | Page is in memory but not mapped in this process’s page table. No disk I/O needed.                       |
| **Major Page Fault**   | Page is **not in RAM** → OS must **load it from disk (swap or file)**. Costly.                |
| **Invalid Page Fault** | Process accessed an **invalid or protected address** → results in segmentation fault (`SIGSEGV`). |

.

🔄 Page Fault Handling Steps (Major Fault):

1. CPU accesses virtual address (e.g., 0x7fff...).
2. MMU → Page Table Lookup → No mapping exists → triggers page fault.
3. CPU raises a page fault exception.
4. OS handles it:

- Checks if it's a valid memory region (e.g., malloc’d, mmap’d).
- if yes → brings the page into RAM (from disk or zero-fill-on-demand).
- Updates page table with new mapping.
- Resumes execution.

**You can see the page fault stats in /proc/[pid]/stat or with vmstat**

> Look at:
> majflt: Major page faults
> minflt: Minor page faults
> ⚠️ Frequent major faults = performance hit (could lead to thrashing).

---------------------

A page fault is a type of exception generated by the processor when a program tries to access a portion of memory that is not currently loaded into physical RAM. In modern operating systems that use virtual memory, that is organized into fixed size blocks called pages, which can be stored either in RAM or on disk. To access data, it must be loaded into RAM. So when a required page is not in RAM, the processor triggers a page fault. This temporarily pauses the running program and invokes a predefined function called an interrupt service routine or ISR. The ISR performs all necessary steps to locate and retrieve the missing page, load it into RAM, update the page tables which track every page in the system, and finally resume the program allowing it to continue running as if nothing ever happened. 

**There are two types of page faults. minor, also known as soft page faults, and major, also known as hard page faults.**

`Minor page` **faults occur when the access data is already in RAM, but hasn't yet been mapped into the virtual memory space of the process. This often happens in cases involving shared memory, such as shared libraries.**<br> `For example,` when a program loads a shared library, it is mapped into the program's virtual memory.<br> However, when other programs access the same library, the data is already in RAM but not yet mapped into<br> their virtual address space. In this case, the operating system simply maps the existing memory to the requesting process.<br> **Minor page folds are typically fast to handle and have little impact on system performance.**

![1749355062259](image/Linux_IPC_socket/1749355062259.png)

`Major page` **faults occur when the requested data is not in RAM and must be loaded from disk<br> before the process can continue.** This usually happens due to demand paging where program pages are<br> loaded into RAM only when accessed or swapping where pages are temporarily evicted from RAM to<br> a designated area on disk called **swap space or swap file**.<br><br> Major page faults take longer to resolve and if they happen frequently in a short time they can significantly degrade system performance. If a program tries to access invalid memory, meaning the address isn't mapped in its virtual address space and **isn't found in either RAM or on disk, the page fault ISR typically asks the operating system to terminate the offending program with the infamous** `segmentation fault error.`

![1749355098190](image/Linux_IPC_socket/1749355098190.png)

--------------
--------------
```
#!/bin/bash

# bash script that lists all currently running processes with their PID, command name, and number of memory pages (total, resident, and shared)

PAGE_SIZE=$(getconf PAGE_SIZE)  # in bytes

printf "%-8s %-25s %10s %10s %10s\n" "PID" "COMMAND" "TOTAL_PAGES" "RES_PAGES" "SHARED_PAGES"

for pid in $(ls /proc | grep -E '^[0-9]+$'); do
    if [ -r /proc/$pid/statm ] && [ -r /proc/$pid/comm ]; then
        read total resident shared _ < /proc/$pid/statm
        cmd=$(cat /proc/$pid/comm)
        # Uncomment it if you want count of pages
        #printf "%-8s %-25s %10s %10s %10s\n" "$pid" "$cmd" "$total" "$resident" "$shared"

        #uncomment it if you want output in MB instead of counts
        printf "%-8s %-25s %10.2f %10.2f %10.2f\n" \
        "$pid" "$cmd" \
        "$(echo "$total * $PAGE_SIZE / 1048576" | bc -l)" \
        "$(echo "$resident * $PAGE_SIZE / 1048576" | bc -l)" \
        "$(echo "$shared * $PAGE_SIZE / 1048576" | bc -l)"

    fi
done

```
---------------
--------------------------
```
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

--------------------------
## Self-contained C program that reads its own memory statistics
Even though your binary might be only 16 KB,
the total mapped memory is much higher — because:

Shared libs like libc and the dynamic loader are mapped.

Kernel reserves stack, heap, and virtual space.

Resident is how much is actually in physical RAM right now.

Shared is memory shared with other processes (usually libraries).

--------------------------
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

### Output (static and Dynamic library linking)

> #Static Library with binary <br> gcc -static mem_map_info.c -o mem_map_info
```c
ninja@ninja:~/workspace$ gcc -static mem_map_info.c -o mem_map_info
ninja@ninja:~/workspace$ ls -lh mem_map_info
-rwxrwxr-x 1 ninja ninja 937K Oct 12 01:00 mem_map_info
ninja@ninja:~/workspace$ ./mem_map_info 
=== Process Memory Summary (/proc/self/statm) ===
Page size          : 4096 bytes
Total pages        : 292 (1168.00 KB)
Resident pages     : 164 (656.00 KB)
Shared pages       : 164 (656.00 KB)
Text (code) pages  : 164 (656.00 KB)
Library pages      : 0 (0.00 KB)
Data + Stack pages : 76 (304.00 KB)

============================================
Page size = 4 KB (standard)
Total pages = 292 → ~1.16 MB virtual address space mapped for this process.
Even though the binary is 937 KB on disk, mapped memory is slightly larger because:
Kernel maps the binary page-aligned → may round up to 4 KB boundaries
Includes .text, .data, .bss segments, heap, stack, etc.
Resident pages = 164 → 656 KB actually loaded in physical RAM.
Shared pages = 164 → all memory is effectively “shared” in this case because there are no separate shared libraries
(everything is inside the binary).
Library pages = 0 → correct, because static linking avoids shared libs.
Data + Stack = 76 pages → heap + stack used by your program.
==============================================

=== Detailed Memory Map (/proc/self/maps) ===
00400000-00401000 r--p 00000000 08:02 2491506                            /home/ninja/workspace/mem_map_info
00401000-004a5000 r-xp 00001000 08:02 2491506                            /home/ninja/workspace/mem_map_info
004a5000-004cd000 r--p 000a5000 08:02 2491506                            /home/ninja/workspace/mem_map_info
004cd000-004d2000 r--p 000cd000 08:02 2491506                            /home/ninja/workspace/mem_map_info
004d2000-004d4000 rw-p 000d2000 08:02 2491506                            /home/ninja/workspace/mem_map_info
004d4000-004da000 rw-p 00000000 00:00 0 
08769000-0878b000 rw-p 00000000 00:00 0                                  [heap]
7126de4fd000-7126de4ff000 r--p 00000000 00:00 0                          [vvar]
7126de4ff000-7126de501000 r--p 00000000 00:00 0                          [vvar_vclock]
7126de501000-7126de503000 r-xp 00000000 00:00 0                          [vdso]
7ffcc74a3000-7ffcc74c5000 rw-p 00000000 00:00 0                          [stack]
ffffffffff600000-ffffffffff601000 --xp 00000000 00:00 0                  [vsyscall]
ninja@ninja:~/workspace$ ^C
ninja@ninja:~/workspace$ 

=============================================
All of these are segments of your static binary:
r-xp → executable code (.text)
r--p → read-only data (.rodata)
rw-p → initialized data (.data)
Last rw-p → .bss (uninitialized data, zero-filled)
Because static binary → all .so code is already inside the binary → no separate /lib or ld-linux mappings.
------------------------------------------------
[heap] → dynamic memory allocated by your program (malloc)
[stack] → process stack
[vdso] → “virtual dynamic shared object” → kernel provides fast userspace system calls (like gettimeofday)
[vvar] & [vvar_vclock] → virtual memory pages providing kernel info for timing and CPU
[vsyscall] → backward compatibility syscall page
Even though statically linked, some special kernel-provided virtual regions still exist.
=============================================

```
**Key Observations**

No separate library mappings → confirms static linking.
`Total pages > file size` → due to page alignment and added heap/stack/kernel mappings.
`Shared pages = resident pages = 164` → all in memory, all “shared” because there are no separate libraries.
`.bss and heap` contribute to extra pages not counted in file size.

> #Dynamic Library<br> gcc mem_map_info.c -o mem_map_info

```c
ninja@ninja:~/workspace$ 
ninja@ninja:~/workspace$ gcc mem_map_info.c -o mem_map_info
ninja@ninja:~/workspace$ ls -lh mem_map_info
-rwxrwxr-x 1 ninja ninja 16K Oct 12 01:11 mem_map_info
ninja@ninja:~/workspace$ 
ninja@ninja:~/workspace$ ./mem_map_info 
=== Process Memory Summary (/proc/self/statm) ===
Page size          : 4096 bytes
Total pages        : 671 (2684.00 KB)
Resident pages     : 343 (1372.00 KB)
Shared pages       : 343 (1372.00 KB)
Text (code) pages  : 1 (4.00 KB)
Library pages      : 0 (0.00 KB)
Data + Stack pages : 90 (360.00 KB)

=== Detailed Memory Map (/proc/self/maps) ===
6019175cd000-6019175ce000 r--p 00000000 08:02 2491506                    /home/ninja/workspace/mem_map_info
6019175ce000-6019175cf000 r-xp 00001000 08:02 2491506                    /home/ninja/workspace/mem_map_info
6019175cf000-6019175d0000 r--p 00002000 08:02 2491506                    /home/ninja/workspace/mem_map_info
6019175d0000-6019175d1000 r--p 00002000 08:02 2491506                    /home/ninja/workspace/mem_map_info
6019175d1000-6019175d2000 rw-p 00003000 08:02 2491506                    /home/ninja/workspace/mem_map_info
60195570e000-60195572f000 rw-p 00000000 00:00 0                          [heap]
7d753ae00000-7d753ae28000 r--p 00000000 08:02 1195860                    /usr/lib/x86_64-linux-gnu/libc.so.6
7d753ae28000-7d753afb0000 r-xp 00028000 08:02 1195860                    /usr/lib/x86_64-linux-gnu/libc.so.6
7d753afb0000-7d753afff000 r--p 001b0000 08:02 1195860                    /usr/lib/x86_64-linux-gnu/libc.so.6
7d753afff000-7d753b003000 r--p 001fe000 08:02 1195860                    /usr/lib/x86_64-linux-gnu/libc.so.6
7d753b003000-7d753b005000 rw-p 00202000 08:02 1195860                    /usr/lib/x86_64-linux-gnu/libc.so.6
7d753b005000-7d753b012000 rw-p 00000000 00:00 0 
7d753b209000-7d753b20c000 rw-p 00000000 00:00 0 
7d753b21f000-7d753b221000 rw-p 00000000 00:00 0 
7d753b221000-7d753b223000 r--p 00000000 00:00 0                          [vvar]
7d753b223000-7d753b225000 r--p 00000000 00:00 0                          [vvar_vclock]
7d753b225000-7d753b227000 r-xp 00000000 00:00 0                          [vdso]
7d753b227000-7d753b228000 r--p 00000000 08:02 1195801                    /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7d753b228000-7d753b253000 r-xp 00001000 08:02 1195801                    /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7d753b253000-7d753b25d000 r--p 0002c000 08:02 1195801                    /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7d753b25d000-7d753b25f000 r--p 00036000 08:02 1195801                    /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7d753b25f000-7d753b261000 rw-p 00038000 08:02 1195801                    /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7ffd00fce000-7ffd00ff0000 rw-p 00000000 00:00 0                          [stack]
ffffffffff600000-ffffffffff601000 --xp 00000000 00:00 0                  [vsyscall]
ninja@ninja:~/workspace$ 

```
