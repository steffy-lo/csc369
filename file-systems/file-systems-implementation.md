# File Systems Implementation

File systems define a block size \(e.g., 4KB\) and disk space is allocated in granularity of blocks.

In order to connect the disk to a computer, a superblock needs to be determined, which represents the root directory

* Always at a well-known disk location
* Often replicated across disk for reliability
* Includes other metadata about the filesystem

A freemap determines which blocks are free

* Usually a bitmap, one bit per block on the disk
* Stored on disk, cached in memory for performance

Data blocks: remaining blocks used to store files and directories

![Green: Superblock, Red: Freemap, White: Data blocks](../.gitbook/assets/image%20%288%29.png)

Disk Layout Strategies - How do you find all the blocks for a file?

1. Contiguous Allocation \(Extent-based\): all blocks of file are located together on disk

![](../.gitbook/assets/image%20%2812%29.png)

Linked \(or chained, structure\): each block points to the next, directory points to the first

![](../.gitbook/assets/image%20%286%29.png)

Indexed Structure \(kind of like address translation

* An "index block" contains pointers to many other blocks
* Mat require multiple, linked index blocks

**Unix inodes** implement an indexed structure for files

* All file metadata is stored in an inode
  * Directory entries map file names to inodes
* Each inode contains 15 block pointers
  * block\[0\]-block\[11\] are direct block pointers
    * Disk addresses of first 12 data blocks in file
  * block\[12\] is a single indirect block pointer 
    * Address of block containing addresses of data blocks
  * block\[13\] is a double indirect block pointer 
    * Address of block containing addresses of single indirect blocks
  * block\[14\] is a triple indirect block pointer

![UNIX inode Example](../.gitbook/assets/image%20%284%29.png)



