# FILE SYSTEMS (through the lens of VSFS)

File systems are pure software constructs. This example of virtualization makes no use of dedicated hardware to boost performance or enhance functionality. 
(from now on file systems will be referred to as FStms)

## Organizaiton
FStms at a very broad level make use of abstract data types and data structures to organize their data and metadata. 
Although file systems can be organized in terms of simple structures such as blocks or singly linked lists, they usually perform bettern with tree-based data structures.

As each disk is divided into blocks, the FS designates a disk region (a number of consecutive blocks) for user data labeled the data region.
Each file's inode collects the metadata relative to it and each inode can be found on the inode table, a data structure. 
Keeping track of inodes and free blocks falls upon the inode and data bitmaps, a simple data structure which indicates whether a given object is free or in use.
Metadata about the FS itself is contained within the superblock.

## Inode
Inode -> index node
Inodes are referred to by i-numbers which are the low-level names of files. Inodes can refer to data blocks through the use of direct or indirect pointers.
Direct pointers are exactly what they sound like.
A multi-level index involves data blocks pointing to other blocks which consist of multiple direct pointers. Indirect pointings can be cascaded to support larger and larger files (2x, 3x, etc.)
Multi-level indeces lead to imbalanced trees, which are based on the assumption that most files are small, and that that's the case which should be optimized in a FS.

## Directories
Just a list of pairs where each entry is formed by a string and a number in the data blocks of the directory. Strings may be variable-length.
Directories are special files and are marked as such in the inode table. 

## Free space management
Free space management is handled through two simple bitmaps.  This is done through the data and inode bitmaps mentioned above.

## Access paths
Accessing any file through a path begins through a root directory, referred to as such because of the tree structure of the FS. This inode has a default value which is easily found by the FS in the inode table.
The directory or file immediately following this one is found through the inode's pointers. If a good reference is found we can repeat the process with the new inode until the desired point is reached.
At this point, if a file is to be read or written to, the appropriate system call sequence is observed: (open(), read() or open(), read(), write(), etc.)
If writes are of concern, before writing the data to the file, the FS must allocate a block for it, and therefore must also modify the structure associated with the blocks.
Each write generates 5 IO requests. File creation involves allocating an inode, allocating blocks within the parent directory.
Caching can be of tremendous help for FS operations, and performance gains are noticeable for unified page caches vs fixed-size caches. 
