# Redundant Arrays of Inexpensive Disks (RAIDs)

The why: Making large, fast and reliable disk?

RAID: using multiple disks in concert to build faster, bigger and more reliable disk system.

RAIDs offer performance gains in capacity, reliability and redundancy through a transparent interface. This in turn makes an RAID more deployable since this reduces worries in software compatibility. 

On the surface, an RAID looks like a disk, i.e. a linear array of blocks to read or write to.
RAIDs take logical I/O requests from the system and perform supplementary calculations to determine the correct disk(s) to access for the requested data and subsequently issue physical I/O request for the appropriate data. 
RAIDs are computer systems, comprised of microcontrollers, processors, memory and disks but which all run on specialized software. 

To handle disk faults, RAIDs  must be aware of certain faults. We can describe these faults through several fault models.
1. Fail-stop model
    Disk is either working or has encountered some error and failed. A failed disk is easily detected and silent failures are non-existent.

RAIDs can be evaluated on the basis of capacity, reliability and performance. 

## RAID-0: Striping
Striping consists in spreading the blocks across disks in a Round Robin fashion. This is done to maximize parallelization and is leveraging spatial locality. The chunk size affects the performance of the array: for instance, small chunck sizes implies more parallelism but longer positioning times for accesses to blocks which span multiple disks, while larger chunk sizes do not suffer from slow positioning times but reduce intra-file parallelism.

Single request latency:latency of a single request to an RAID helps us understand the level of parallelism can exist in a single logical I/O operation.

Steady state throughput: Total bandwith of many concurrent requests, which helps us determine suitability for high-performance use cases. 

Two types of workloads help determine the throughput in requests: sequential and random. 
Sequential workloads assume requests come in contiguous, large chunks. 
Random workloads assume requests are rather small and target different areas on disk. 

The throughput is equal to the number of requests multiplied by the sequential or random bandwidth of the workload in question.
Single request latency: S or R
Steady state throughput = N * S or N * R

## RAID-1: Mirroring
Mirroring boils down to making at least a copy of each block on the system, where each copy lives on a separate disk. This is a fault-tolerant system with regards to disk failures. It can be assumed that each logical block has another physical copy on a different disk. Mirroring can be combined with striping where each disk is striped and has another striped copy.
Since creating a copy of every block is not efficient, mirroring is wasteful in terms of capacity, with only a useful capacity of N * B / 2
(while at the same time being tolerant of fault-failures). Reads are the same in terms of bandwidth as RAID-0, but each logical write request results in two physical writes (with poor seek and rotational delay).
Sequential read latency: (N * S) / 2
Steady State throughput: (N * R) / 2

## RAID-4: Saving Space with Parity
Parity seeks to improve on mirroring through the storage of redundant information on specific disks within the RAID.
Each row within a RAID must have an even number of 1s (XOR must result in a value of 0). Each block can't be XORed with other blocks so we use the result of the block's XOR as input for the parity XOR.
Useful capacity in RAID-4 is (N - 1) * B, since there is one parity disk for every group of blocks.
Single request latency: (N - 1) * S or (N - 1) * R
Steady state throughput: (R / 2)

## RAID-5: Rotating Parity
Similar to RAID-4 but parity blocks are rotated across drives. 
Useful capacity : Same as RAID-4
Sequential read latency: Same as RAID-4
Single request latency: R or S for reads, 2 * R or 2 * S for writes
Total bandwidth for small writes: (N * R) / 4

# Files and Directories
