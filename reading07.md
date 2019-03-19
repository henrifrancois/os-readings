# Reading 07 

## Swapping Mechanisms

Given that real address spaces do not fit into physical memory, we need new mechanisms to handle large address spaces too bit to fit into physical memory. 

Large address spaces require the use of slow memory (hard disk) which is larger, but much slower then physical memory. This is why we need some new levels in the memory hierarchy, to properly make use of the slower memory while mitigating its pitfalls as much as possible.

Swap space is defined as the space reserved in the hard disk for pages. Each page is associated with a disk address which the OS reads to swap pages into disk or swap pages out of disk. 

The present bit is an additional hardware-related measure which signals the presence or absence of the requested page in memory after a TLB miss. If the present bit is set to one, then the page exists in memory, and there isn't really an issue moving forward, but in the opposite case, we encounter a page fault and the requested page must be fetched from disk. 

A page fault handler is an OS process which takes over iff a page fault occurs. The page fault handler looks into the page table to find the location of a page in disk, specifically into each PFN to determine an address. Then it issues a request for the page at that specific address, and sets the present bit to 1. 

Cases where memory is full, implying the need for page replacements, are handled by page-replacement policies. 
A complete description of this 

The OS actually keeps a small portion of memory free, as it is advantageous to do so:
high and low watermark determine thresholds for replacement operations. The monitoring and subsequent actions stemming from the state of the LW rests upon the swap daemon, a separate thread dedicated to what its name clearly implies. 
This can be done for a number of pages at once, which is somewhat more efficient. 




## Swapping Policies
Main memory can be thought of as a cache for virtual memory. 
We can create replacement policies which aim to minimize cache misses, since even small miss rates result in significant growth of the AMAT. 

The optimal page replacement policy being _furthest in the future_, where the page kicked out is the one which will be used the furthest in the future. This isn't necessary the best choice in real systems as the future is hard to determine.

Several paging policies can be used:

**FIFO**
Simple fifo replacement policies remove the page placed at the end of a queue, which was the first page to enter the queue.
Poort hit rate.

**Random replacement**
Self explanatory

**LRU (least recently used)**
Recency of access is used to determine which page to kick out, exploiting locality. Thus a page is most likely to be kicked out when it hasn't been accessed in a long period of time. 

**Performance and workloads**
LRU performs much better with workloads exhibiting high locality, while for workloads exhibiting little to no locality, it performs similarly to other policies, or better. 

**Implementation concerns**
Tracking page accesses is cumbersome, but it can be made easier with hardware support, which demonstrates how difficult implementing LRU perfectly is. (Millions of pages would imply a really, really large data structure)
We can avoid some of the complexities of perfect implementation through some approximation. With hardware support, we can assign a use bit which can't be cleared by hw for each page, then for each replacement situation the use bit indicates if a page is a candidate for replacement. If it isn't the OS clears the bit and keeps going until it finds the next clear bit, and replaces the corresponding page. This is the clock algorithm. 
The clock algorithm can be improved to take into consideration pages that have been modified to consider modified pages. With hardware we can define a dirty bit which tracks whether a page has been modified (writing modified pages to disk is expensive, while there is no cost for unmodified pages). Then priority for replacement can be given to unused and clean pages, then used and clean pages, then used and dirty pages, etc. 


Constant paging is defined as thrashing, which results in abysmal performance. Admission control, which only allocates runtime to processes the OS knows it can handle well, is a remedy to thrashing. Another approach is to just kill processes that use too much memory
