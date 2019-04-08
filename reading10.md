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
Striping consists in spreading the blocks across disks in a Round Robin fashion. This is done to maximize parallelization and is leveraging spatial locality. The chunk size determines 

## RAID-1: Mirroring

## RAID-4: Saving Space with Parity

## RAID-5: Rotating Parity


# Files and Directories
