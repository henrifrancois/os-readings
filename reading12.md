# Reading 12

## Fast File System
A simple layout for a file system can be good from the perspective of a developer, but can incur some serious performance costs because of poor
decisions with regards to the underlying structure of the file system. Fragmentation in particular is one such issue that affects systems such 
as the old Unix file system.

By making file system operations "disk-aware", there are significant performance gains to be achieved.

### Disk-aware optimizations
1. Cylinder groups: files are placed in 'cylinders' within the same group to minimize seek times.
Items related to each other are placed in the same regions on disk.

## FSCK and Journaling
What should happen once the power runs out, either due to a system crash or a shutdown?

The file system checker (fsck) is a tool used to fix inconsistencies resulting from crashes & co. It is a tool running before the file system is
mounted and makes use of the superblock and of the inode and data bitmaps to decide on appropriate fixes to the file system. By verifying the
states of the various data structures, This approach is, however, too slow. There exist different approaches can be better for the problem at hand. 

Journaling (write ahead logging)
This idea consists in keeping logs of future actions which can then be recovered in the case of a system failure to execute those previously planned
actions. Journaling can be divided into physical or logical varieties: physical logging writes the entire content of a planned operation in the 

## Log-Structured File System

### Choice of block to clean
A hot segment is one in which the contents are being frequently over-written; thus, for such a segment, the best policy is to wait a long time
before cleaning it, as more and more blocks are getting over-written (in new segments) and thus being freed for use

### Crash Recovery and the Log
During normal operation, LFS buffers writes in a segment, and then (when the segment is full, or when some amount of time has elapsed), writes
the segment to disk. LFS organizes these writes in a log,
Because LFS writes the CR every 30 seconds or so, the last consistent snapshot of the file system may be quite old. Thus, upon reboot, LFS can
easily recover by simply reading in the checkpoint region, the imap pieces it points to, and subsequent files and directories; however, the last
many seconds of updates would be lost.

LFS introduces a new approach to updating the disk. Instead of overwriting files in places, LFS always writes to an unused portion of the disk,
and then later reclaims that old space through cleaning. This approach, which in database systems is called shadow paging [L77] and in file-system-speak
is sometimes called copy-on-write, enables highly efficient writing, as LFS can gather all updates into an in-memory segment and then write them
out together sequentially.
