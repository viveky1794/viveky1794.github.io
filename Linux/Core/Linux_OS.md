# What is OS?

OS is a program which manages computer resources while serving programs.
Suppose there is not an OS then every program Developer need to write its resource management code also.
In such case every Application became bulky due to duplicate code.
Which is clearly violation of **DRY**(Don't repeat yourself) laws.
OS provides **isolation & Protection**.

# Types of OS


# Difference between Multi-Processing, Multi-Tasking and Multi-Threading ??

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