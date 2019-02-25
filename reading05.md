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

> How to correctly and efficiently allocate and manage memory?

### Types of Memory

There are two main types of memory:
1. **Stack or automatic memory**
   The stack encompasses variables declared by the programmer, functions, etc.
   Information living on the stack does not persist across different scopes.
   Allocations and deallocations are implicitely managed by the compiler.
2. **Heap or manual memory**
   Memory allocations on the heap are explicitly handled by the programmer.
   Information living on the heap persists across different scopes.
   
### `malloc()`
The `malloc(size_t size)` call returns a pointer to newly allocated space (with size of size) on the heap on success or `NULL` on failure.

Common malloc patterns:
```c
double *d = (double *) malloc(sizeof(double));
```
Allocating memory for a certain type, here double. Casting is not required, but is used for readability.

```c
int *x = malloc(10 * sizeof(int));
printf("%d\n", sizeof(x));
```
The actual amount of memory allocated to x is not known at compile time with the use of sizeof.

```c
char * s = "some string";
char * str2 = (char *) malloc(strlen(s) + 1);
```
When allocating memory for strings, it is necessary to make space for the string termination character (`\0`), hence the `strlen(s) + 1`.

### `free()`
The `free((void*) arg)` call removes allocated memory for the variable `arg` from the heap.
Only free when `arg` has accomplished its mission and will no longer be in use.

### Common memory errors

#### Forgetting to allocate memory
```c
char *src = "hello";
char *dst; // oops! unallocated
strcpy(dst, src); // segfault and die
```

#### Insufficient allocations
```c
char *src = "hello";
char *dst = (char *) malloc(strlen(src)); // too small!
strcpy(dst, src); // work properly
```

#### Uninitialized allocated memory
```c
int *p = (int *)malloc(sizeof(int));
...
func(p, ...); // uninitialized read
```

#### Not freeing allocating memory
Memory leak: allocated memory is never freed. Memory leaks lead to insufficient space in the head for the rest of the program. 

#### Freeing memory which will be referenced
Dangling pointer: memory has been freed but should be used again. 

#### Freeing the same memory multiple times
Double free: freeing the same portion twice. 

#### Incorrect call to free
Freeing something that was not malloc()ed. 

# Chapter 15: Address Translation

> How to efficiently and flexibly virtualize memory?

Every memory reference in code goes through a hardware-based address translation, which enables the virtual memory reference to map to a physical memory address, where the data actually lives. 
In hardware-based address translation, the OS manages memory, keeping track of free locations and locations which are in use to provide the low-level mechanisms for the hardware to perform the translation.

## Assumptions
1. User's address space is placed contiguously in physical memory
2. The size of the address space is smaller than that of the physical memory
3. All address spaces are the same size

## Dynamic Relocation (Base and bounds)
Each CPU harbors a base register and a bounds (or limits) register.
The CPU uses the value of the base register to define the starting point for the process' virtual address space. For instance, if the CPU loads a process at address 32k, then it has to set the base register to that value. 
Each virtual address referenced by the running process is determined by this formula:
`physAdrr = virtualAddr + Base`
THe bounds register defines the limits of the address space. If an invalid address (value > bounds) is presented to the CPU for translation, the CPU raises a runtime exception and kills the offending process.

## Operating System Issues
The Operatings System holds responsibility for a correct address translation. At the time of the creation of a process, the OS must find space for it in memory. It parses a **free list** to find room for new address space, and then mark it used. 
In addition, when a process is terminated or exits gracefully, the OS must place it's memory on the free list, and clean up any associated data structures spawned by the program.
Because the base and bounds registers' values are process-defined, a context switch implies new values for both registers. Therefore, during a context switch, the OS must also save both registers to the PCB, and restores them when the process is scheduled to run again.
Finally, the OS must provide exception handlers, for a plethora of possible unallowed behaviors, such as attempts to access memory which does not belong to the offending process, or running illegal instructions. 

# Chapter 16: Segmentation

> How to support a large address space, given the limits of the base and bounds approach?

## Segmentation: Generalized base and bounds
Programs with large address spaces would be quite problematic in our initial model, since they would be engulfing a significant portion of memory each time they would be running.
**Segmentation** is the idea that the different portions of memory in a program (code, stack, heap), can be logically separated within the address space. Additionally, segmentation implies that the initial memory segments loaded into the address space are not exactly contiguous: space is left for each of these different segments to grow individually. Each segment is defined by a base and bounds registers. The address translation for each segment is done the same as before. When an out-of-bounds address is referred to, a **segmentation fault** occurs.

## Segment References
Segment bounds can be explicitly defined, where the bounds register is assigned a value by default, or implicitly defined bounds, where the hardware determines the bounds based on how the address was formed.

## Stack
Since the stack grows upwards instead of downwards, the MMU needs information on the direction of growth of a segment. Then the address translation is done based on the direction of growth of the segment. 

## Support for Sharing
Code sharing is a thing, but _protection bits_ are needed to make it happen. Protection bits indicate the permissions (read, write, execute) by a program on a specific segment. 

## Fine-grained vs Coarse-grained Segmentation
We've covered coarse-grained segmentation, but fine-grained segmentation is more flexible in terms of memory address spaces, but require additional hadware support, such as a segment table 

## OS Support
External fragmentation describes the state of memory containing a number of small ranges of free space but lacking adequate continuous free space range to accomodate new segments. While it is possible to rearrange the available space to accomodate the incoming segment, it is an expensive operation. Free list management algorithms may help keep large extents of memory available for allocation. As this is a hard problem, there isn't really an optimal solution ¯\_(ツ)_/¯
