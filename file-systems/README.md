# File Systems

* Provide long-term information storage
* Requirements:
  * Store very large amounts of information
  * Information must survive the termination of process using it
  * Multiple processes must be able to access info concurrently
* Two views of file systems:
  * User view – convenient logical organization of information
  * OS view – managing physical storage media, enforcing access restrictions

### File Management Systems

* Implement an abstraction \(files\) for secondary storage 
* Organize files logically \(directories\)
* Permit sharing of data between processes, people, and machines • Protect data from unwanted access \(security\)

### File Operations

* Creation
  * Find space in file system, add entry to directory
  * map file name to location and attributes
  * By default, a file would result in the creation of two directories "." and "..", corresponding to itself, and the parent directory
* Writing
* Reading
  * Dominant abstraction is “file as stream”
* Repositioning within a file
* Deleting a file
* Truncation and appending
  * May erase the contents \(or part of the contents\) of a file while keeping attributes

### File Access Methods

* General-purpose file systems support simple methods
  * Sequential access – read bytes one at a time, in order
  * Direct access – random access given block/byte number
* Database systems support more sophisticated methods
  * Record access - fixed or variable length
  * Indexed access

### Directories

Directories provide logical structure to file systems 

* For users, they provide a means to **organize files**
* For the file system, they provide a **convenient naming interface**
  * Allows the implementation to separate logical file organization from physical file placement
  * Stores information about files \(owner, permission, etc.\)

![Multi-Level Directories](../.gitbook/assets/image%20%2829%29.png)

### Directory Structure

* A directory is a list of entries - names and associated metadata 
  * Metadata is not the data itself, but information that describes properties of the data \(size, protection, location, etc.\)
* List is usually unordered \(effectively random\)
  * Entries usually sorted by program that reads directory 
* Directories typically stored in files 
  * Only need to manage one kind of secondary storage unit

### Directory Implementation

1. Single-level, two-level, or tree-structured
2. Acyclic-graph directories: allows for shared directories
   * The same file or subdirectory may be in 2 different directories

![](../.gitbook/assets/image%20%2814%29.png)

Data Structures

Option 1: List

* Simple list of file names and pointers to data blocks
* Requires linear search to find entries
* Easy to implement, slow to execute
  * And directory **operations are frequent!**

Option 2: Hash Table 

* Create a list of file info structures
* Hash file name to get a pointer to the file info structure in the list
* Hash table takes space

### File Links

Sharing can be implemented by creating a new directory entry called a link: a pointer to another file or subdirectory

* Symbolic, or soft, link
  * Directory entry refers to file that holds “true” path to the linked file
  * Works essentially like a shortcut link
* Hard links
  * Second directory entry identical to the first
  * Mostly for convenience

![](../.gitbook/assets/image%20%2816%29.png)

**Issues with Links**

* With links, a file may have multiple absolute path names
  * Traversing a file system should avoid traversing shared structures more than once
* Maintaining consistency is a problem
  * How do you update permissions in directory entry with a hard link?
* Deletion: When can the space allocated to a shared file be deallocated and reused?
  * Somewhat easier to handle with symbolic links
    * Deletion of a link is OK; deletion of the file entry itself deallocates space and leaves the link pointers dangling
  * Keep a reference count for hard links
* Sharing: How can you tell when two processes are sharing the same file?

**Summary: File System Goals**

* Efficiently translate file name into file number using a directory
* Sequential file access performance
* Efficient random access to any file block
* Efficient support for small files \(overhead in terms of space and access time\)
* Support large files
* Efficient metadata storage and lookup
* Crash recovery

**Summary: File System Components**

* Index structure to locate each block of a file
* Free space management
* Locality heuristics
* Crash recovery

