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

![Green: Superblock, Red: Freemap, White: Data blocks](../.gitbook/assets/image%20%2816%29.png)

Disk Layout Strategies - How do you find all the blocks for a file?

1. Contiguous Allocation \(Extent-based\): all blocks of file are located together on disk

![](../.gitbook/assets/image%20%2822%29.png)

Linked \(or chained, structure\): each block points to the next, directory points to the first

![](../.gitbook/assets/image%20%2813%29.png)

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

![UNIX inode Example](../.gitbook/assets/image%20%2811%29.png)

### Overall Organization

We now develop the overall on-disk organization of the data structures of the vsfs \(Very Simple File System\) ﬁle system. The ﬁrst thing we’ll need to do is divide the disk into blocks; simple ﬁle systems use just one block size \(e.g., 4 KB\). The blocks are addressed from 0 to N − 1, in a partition of size N 4-KB blocks.

![VSFS Disk Layout](../.gitbook/assets/image%20%288%29.png)

D \(User Data\)

* Where most of the disk is used to store user data, leaving a little space for storing other things like metadata

I \(Inode Table\)

* Needs to track information about each file \(i.e., metadata\)
* The info of each file is kept in a struct called inode
* In ext2, each inode has a size of 128 B, this means that in our VSFS, the maximum number of files it can hold it 2 \* 4KB / 128 KB = 64 files

DB \(Datablock Bitmap\)/IB \(Inode Bitmap\)

* Keep track of which block are being used and which ones are free \(0 is free and 1 is in-use\)
* A bitmap for the data region and a bitmap for the inode region
  * Reserve one block for each bitmap

S \(Superblock\)

* Contains information about this particular file system: what type of file system it is \(“VSFS” indicated by a magic number\)
  * how many inodes and data blocks are there \(160 and 56\) where the inode table begins \(block 3\), etc.
* When mounting a file system, the OS first reads the superblock, identifies its type and other parameters, then attaches the volume to the file system tree with proper settings

Example: Read a file with inode number 30

* From the superblock, we know 
  * inode table begins at Block 3, i.e., 12KiB \(at byte 12,288\) 
  * inode size is 128B 
* Calculate the address of inode 30 
  * 12,288 + 30 \* 128 = 16,128 \(Block 4 starts at byte 16384\)

### Imbalanced Tree

* Multi-Level index approach to file pointers
* Why?
  * Most files are small
  * Files are usually accessed sequentially
  * Directories are typically small \(20 or fewer entries\) 

### Another Approach: Extent-Based

* An extent is a disk pointer plus a length \(in \# of blocks\), i.e., allocates a few blocks in a row
* Instead of requiring a pointer to every block of a file, we just need a pointer to every several blocks \(every extent\)
* Disadvantage: less flexible than the pointer-based approach \(external fragmentation?\)
* Advantages: uses smaller amount of metadata per file, and file allocation is more compact
* Adopted by ext4, HFS+, NTFS, XFS

### Yet Another Approach: Link-Based

* Instead of pointers to all blocks, the inode just has one pointer to the first data block of the file, then the first block points to the second block, etc.
* Works poorly if we want to access the last block of a big file
* Use an in-memory File Allocation Table, indexed by address of data block
  * Faster in finding a block
* FAT file system, used by Windows by NTFS

### Unix Inodes and Path Search

* Unix inodes are not directories
* They describe where on the disk the blocks for a file are placed
  * Directories are files, so inodes also describe where the blocks for directories are placed on the disk
* Directory entries map file names to inodes

### Operations

* mkdir\(“/x”\) - Create a directory
* creat\(“/x/y”\) - Create an empty file
* unlink\(“/x/y”\) - Remove a file or directory
* fd = open\(“/x/z”, O\_CREAT \| O\_WRONLY\); 
* write\(fd, buf, BLOCKSIZE\); close\(fd\); 
  * Open a file for writing, and write one block to it
  * If it does not exist, create it

