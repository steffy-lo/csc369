# Disk I/O File System Optimization

![](../.gitbook/assets/image%20%2815%29.png)

### Disk Performance

* Disk request performance depends on a number of steps
  * Seek - moving the disk arm to the correct cylinder \(most expensive\)
    * Depends on how fast disk arm can move
    * Typical times: 1-15ms, depending on distance \(avg 5-6ms\)
  * Rotation - waiting for the sector to rotate under the head
    * Depends on rotation rate of disk
    * Average latency of 1/2 rotation
  * Transfer - transferring data from surface into disk controller electronics, sending it back to the host
    * Depends on density \(increasing quickly\)
    * Improving rapidly \(~40% per year\)

![](../.gitbook/assets/image%20%2817%29.png)

* When the OS uses the disk, it tries to minimize the cost of all of the above steps
  * Particularly seeks and rotation

![Track Skew](../.gitbook/assets/image%20%281%29.png)

* If the arm moves to outer track too slowly, may miss sector 36 and have to wait for a whole rotation
* Instead, skew the track location, so that we have enough time to position

![Zones](../.gitbook/assets/image%20%2827%29.png)

* The density of inner track is larger
* Outer tracks are larger by geometry, so they should hold more sectors

### Cache: Track Buffer

* A small memory chip, part of the hard drive \(usually 8-16 MB\)
* Different from OS cache
  * Unlike the OS cache, it is aware of the disk geometry
  * When reading a sector, may cache the whole track to speed up future reads on the same track

### Disks and the OS

* Disks are messy physical devices:
  * Errors, bad blocks, missed seeks, etc.
* The job of the OS is to hide this mess from higher level software
  * Low-level device control \(initiate a disk read, etc.\)
  * Higher-level abstractions \(files, databases, etc.\)
* The OS may provide different levels of disk access to different clients
  * Physical disk \(surface, cylinder, sector\)
  * Logical disk \(disk block \#\)
  * Logical file \(file block, record, or byte \#\)

### Disk Scheduling

* Because seeks are so expensive, the OS tries to schedule disk requests that are queued waiting for the disk
* In general, unless there are request queues, disk scheduling does not have much impact; important for servers, less so for PCs
* Modern disks often do the disk scheduling themselves. Disks know their layout better than OS, and can optimize better
  * FCFS \(First Come First Serve\)
    * Reasonable when load is low
    * Long waiting times for long request queues
  * SSTF \(Shortest Seek Time First\)
    * Minimize arm movement \(seek time\), maximize request rate
    * Favours middle blocks
  * SCAN \(Elevator\)
    * Service requests in one direction until done, then reverse
  * C-SCAN
    * Like SCAN, but only go in one direction \(typewriter\)
  * LOOK/C-LOOK
    * Like SCAN/C-SCAN but only go as far as last request in each direction \(not full width of disk\)

### Enhancing Disk Performance

* High-level disk characteristics yield two goals:
  * Closeness 
    * Reduce seek times by putting related things close to one another
    * Generally, benefits can be in the factor of 2 range
  * Amortization
    * Amortize each positioning delay by grabbing lots of useful data
    * Generally, benefits can reach into the factor of 10 range
* Allocation Strategies
  * Disks perform best if seeks are reduced and large transfer are used
  * Scheduled requests is one way to achieve this
  * Allocating related data "close together" on the disk is even more important

![](../.gitbook/assets/image%20%2848%29.png)

* Pro: File size grows dynamically, allocations are independent
* Con: Hard to achieve closeness and amortization

### FFS: A Disk-Aware File System

Original Unix File System

* Sees storage as linear array of blocks
* Each block has a logical block number \(LBN\)
* Simple, straightforward implementation, but very poor utilization of disk bandwidth \(lots of seeking\)

![Data and Inode Placement: Problem 1](../.gitbook/assets/image%20%2811%29.png)

![Data and Inode Placement: Problem 2](../.gitbook/assets/image%20%2824%29.png)

* BSD Unix folks did a redesign \(BSD 4.2\) that they called the Fast File System \(FFS\) 
  * Improved disk utilization, decreased response time
* Addressed placement problems using the notion of a cylinder group \(aka allocation groups in lots of modern FS's\)
  * Disk partitioned into groups of cylinders
  * Data blocks in same file allocated in same cylinder group
  * Files in same directory allocated in same cylinder group
  * Inodes for files allocated in same cylinder group as file data blocks

![](../.gitbook/assets/image%20%288%29.png)

* Allocation in cylinder groups provides closeness
  * Reduces number of long seeks
* Free space requirement
  * To be able to allocate according to cylinder groups, the disk must have free space scattered across cylinders --&gt; 10% of the disk is reserved just for this purpose
  * When allocating a large file, break it into large chunks and allocate from different cylinder groups, so it does not fill up one cylinder group
  * If preferred cylinder group is full, allocate from a "nearby" group
* Small blocks \(1K\) in original Unix FS caused 2 problems:
  * Low bandwidth utilization
  * Small max file size \(function of block size\)
* Fix using a large block \(4K\)
  * Very large files, only need two levels of indirection for 2^32
  * New problem: internal fragmentation
  * Fix: introduce "fragments" \(1K pieces of a block\)
* Problem: Media failures
  * Replicate master block \(superblock\)

### NTFS \(New Technology File System\)

* Replaced the old FAT system
* The designers had the following goals:

  1. Eliminate fixed-size short names
  2. Implement a more thorough permissions scheme
  3. Provide good performance
  4. Support large files
  5. Provide extra functionality:
     * Compression
     * Encryption
     * Types

  \*\*\*Wanted a file system flexible enough to support future needs

* Each volume \(partition\) is a linear sequence of blocks \(usually 4KB block size\)
* Each volume has a Master File Table \(MFT\)
  * Sequence of 1 KB records
  * One or more record per file or directory \(similar to inodes, but more flexible\)
  * Each MFT record is a sequence of variable length \(attribute header, value\) pairs
  * Long attributes can be stored externally, and a pointer kept in the MFT record
* NTFS tries to allocate files in runs of consecutive blocks

![MFT Record](../.gitbook/assets/image%20%2845%29.png)

* An MFT record for a 3-run 9-block file
* Each "data" attribute indicates the starting block and the number of blocks in a "run" \(or extent\)
* If all the records don't fit into one MFT record, extension records can be used to hold more

![MFT Record For A Small Directory](../.gitbook/assets/image%20%2850%29.png)

* Directory entries are stored as a simple list
* Large directories use B+ trees instead

![MFT For A Small File](../.gitbook/assets/image%20%2813%29.png)

* For very small files, data can be stored in the MFT record

## Supporting Multiple File Systems

### Virtual File System \(VFS\)

* Provides an abstract file system interface
  * Separates abstraction of file and collections of files from specific implementations
  * System calls such as open, read, write, etc. can be implemented in terms of operations on the abstract file system \(e.g., vfs\_open, vfs\_close\)
* Abstraction layer is for the OS itself
  * user-level programmer interacts with the file systems through the system calls

![Schematic View of VFS](../.gitbook/assets/image%20%2838%29.png)

