# Complete Virtual Memory Systems


## VAX/VMS Virtual Memory
### Memory Management HW
32-bit address space with 23 bit VPN, 9 bit offset, 512 byte pages and first two bits of VPN reserved to distinguish pages. 
The lower half of the address space was designated as the process space and was unique to each process. Primarily, small pages implied large page table sizes, which could overwhelm memory. This was to be avoided. 
The first method to reduce page table impact on memory was by segmenting the user space for each process into two.
The second method worked by placing user page tables in kernel virtual memory.

### A real address space
The 0 address space is used to detect null pointer accesses. 
Kernel virtual address space is a part of each user address space. On a contextswitch, the OS changes the P0 and P1 registers to point to the appropriate page tables of the soon-to-be-run process; however, it does not change the S base and bound registers, and as a result the “same” kernel structures are mapped into each user address space.
OS protection through the use of privilege levels specified in the protections bits of the page table. 

### Page replacement
Because there is no reference bit in the PTE and to avoid overuse of memory, VAX made use of the segmented FIFO replacement policy, which keeps each process' in-memory pages in a queue. When a process exceeds its resident set size, the first page is evicted to a second chance list, specifically in the clean page free list or the dirty page list. 

### Other neat tricks
VMS had two other now-standard tricks: demand zeroing and copy-onwrite. 
* Demand zeroing, the OS instead does very little work when the page is added to your address space; it puts an entry in the page table that marks the page inaccessible. If the process then reads or writes the page, a trap into the OS takes place. When handling the trap, the OS notices (usually through some bitsmarked in the “reserved for OS” portion of the page table entry) that this is actually a demand-zero page; at this point, the OS does the needed work of finding a physical page, zeroing it, and mapping it into the process’s address space. If the process never accesses the page, all suchwork is avoided, and thus the virtue of demand zeroing.
* Copy on write: when the OS needs to copy a page from one address space to another, instead of copying it, it can map it into the target address space and mark it readonly in both address spaces. If both address spaces only read the page, no further action is taken, and thus the OS has realized a fast copy without actually moving any data.


## Linux Virtual Memory System
### Linux Address Space
Linux addr spaces contain user and kernel portions. The split between user and kernel portions of the address space takes
place at address 0xC0000000. There are two different kinds of kernel virtual addresses: kernel virtual addresses and kernel logical addresses. KLAs are the normal virtual address space and we can get more out of it by calling kmalloc.

### Page Table Structure
More recently, as the need for better TLB behavior is more commonamongmany applications, Linux developers have added transparent huge
page support. When this feature is enabled, the operating system automatically looks for opportunities to allocate huge pages without requiring application modification.

### Large Page Support
Huge pages are not without their costs. The biggest potential cost is internal fragmentation, i.e., a page that is large but sparsely used. This form of waste can fill memory with large but little used pages.

### The page cache
To reduce costs of accessing persistent storage (the focus of the third part of this book), most systems use aggressive caching subsystems to keep popular data items in memory. Linux, in this regard, is no different than traditional operating systems.
The Linux page cache is unified, keeping pages in memory from three primary sources: memory-mapped files, file data and metadata from devices and heap and stack pages that comprise each process.
These entities are kept in a page cache hash table, allowing for quick lookup when said data is needed.

### Security and Buffer Overflows
One major threat is found in buffer overflow attacks, which can be used against normal user programs and even the kernel itself. The idea of these attacks is to find a bug in the target systemwhich lets the attacker inject arbitrary data into the target’s address space.
