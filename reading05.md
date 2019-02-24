# Reading 05
## Memory Overview


# Chapter 13: Address Spaces

In a multiprogrammed OS, letting each running process access the entirety of physical memory presents the issue:
+ Writing the state of each process to disk before a context switch is extremely slow.

If, in a time-shared OS, we prefer to let each process reside in memory instead of storing its state to disk:
+ there must exist a way for each process' memory space to be private (inaccessible by other processes)

### How can the operating system provide each process a _private, large_ memory space?

Introducing the **_address space_**, an abstraction for the physical memory which enables each process to _"have"_ a private, relatively large memory space. 

### How does virtual memory work? 

By providing each process with a  virtual memory space, the process' stack and heap are able to grow without the complexities associated with physical memory. Virtual and physical memory addresses are not in reality the same. Each virtual memory address refers to a specific physical memory address. 

### What does memory virtualization hope to achieve?

1. Transparency
   This implies a complete decoupling between the memory virtualization procedure and any running thread.
2. Efficiency
   Memory virtualization should be as efficient as possible, which is likely reliant on hardware support. 
3. Protection
   Processes remain isolated from each other, and virtual memory spaces of different processes never overlap. 

# Chapter 14: Memory API

# Chapter 15: Address Translation

# Chapter 16: Segmentation
