# Solid State Drives \(SSDs\)

* Replace rotating mechanical disks with non-volatile memory 
  * Battery-backed RAM
  * NAND flash
* Advantages: faster
* Disadvantages: 
  * Expensive
  * Wear-out \(flash-based\)
* NAND flash storage technology 
  * Read / write / erase! operations

### Characteristics

* Data cannot be modified "in place"
  * No overwrite without erase
* Terminology
  * Page \(unit of read/write\), block \(unit of erase operation\)
* Uniform random access performance
  * Disks typically have multiple channels so data can be split \(striped\) across blocks, speeding access time

### Writing

* Consider updating a file system block \(e.g., a bitmap allocation block in ext2 file system\)
  * Find block containing the target page
  * Read all active pages in the block into controller memory
  * Update target page with new data in controller memory
  * Erase the block \(high voltage to set all bits to 1\)
  * Write entire block to drive
* Some file system blocks are frequently updated
  * And SSD blocks wear out \(limited erase cycle\)
* Log-Structured: Upon a write to logical block N, the device appends the write to the next free spot in the currently-being-written-to block
* To allow for subsequent reads of block N, the device keeps a **mapping table** \(in its memory, and persistent, in some form, on the device\); this table stores the physical address of each logical block in the system.

### SSD Algorithms

* Wear Levelling: spread writes across the blocks of the ﬂash as evenly as possible, ensuring that all of the blocks of the device wear out at roughly the same time
  * Always write to a new location
  * Keep a map from logical FS block number to current SSD block and page location
  * Old versions of logically overwritten pages are "stale"
* Garbage Collection
  * Reclaiming stale pages and creating empty erased blocks
* RAID 5 \(with parity checking\) striping across I/O channels to multiple NAND chips

