
# Process Table

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

<font color = 'orange'>**Process Table Image**</font>

![Process_Table](./images/Process_Table.png)



# Process Control  Block 

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









