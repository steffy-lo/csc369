# Virtual Memory

![](.gitbook/assets/image%20%2865%29.png)

But physical memory is more complicated...

* It doesn't store your code starting a physical address 0
* 2^32 is 4GB so how did we run processes on 32-bit machines in the days when we had less than 1 GB of physical memory?

![](.gitbook/assets/image%20%286%29.png)

### Virtualizing Memory Goals

* Transparency
  * A program should not need to know how memory is virtualized
  * Each program has the illusion of using all of physical memory
* Efficiency
  * Time - programs shouldn't run much more slowly
  * Space - minimize the size of structures needed to support virtual memory
* Protection
  * A process should not be able to access another process memory: isolation

### Transparency

* Give each process its own abstract view of memory
  * Large contiguous address space, starting at address 0
  * Simplifies memory allocation
* Decouple the data layout from where the data is actually stored in physical memory



* Virtual to Physical: how do we take a process' abstract \(logical\) view of memory and translate it to a region in physical memory?
* Assume a process' memory is stored in physical memory in one contiguous chunk. How can we arrange these chunks in physical memory?

![](.gitbook/assets/image%20%2869%29.png)

![](.gitbook/assets/image%20%284%29.png)

![Placement Example](.gitbook/assets/image%20%2874%29.png)

![](.gitbook/assets/image%20%2811%29.png)

## Dynamic Partitioning

![](.gitbook/assets/image%20%285%29.png)

### Placement Heuristics

* Compaction is time-consuming and not always possible
* We can reduce the need for it by being careful about how memory is allocated to processes over time
  * First-Fit: choose first block that is large enough; search can start at beginning, or where previous search ended \(Next-Fit\)
    * Simplest, often the fastest and most efficient
    * May leave many small fragments near start of memory that must be searched repeatedly
      * Next-Fit variant allocates from end of memory; free space becomes fragmented more rapidly
  * Best-Fit: choose the block that is closest in size to the request
    * Leftover fragments tend to be small \(unusable\)
    * In practice, similar storage utilization to first-fit
  * Worst-Fit: choose the largest block
  * Quick-Fit: keep multiple free lists for common block sizes
    * Great for fast allocation, generally hard to coalesce

### Requirements

* Relocation
  * Programmer's don't know what physical memory will be available when their programs run
  * May swap processes in/out of memory, need to be able to bring it back in to a different region of memory
  * Implies that there is a need for some type of address translation
* Protection
  * A process' memory should be protected from unwanted access by other processes, both intentional and accidental
  * Requires hardware support

### Problems With Partitioning...

Basic problem is assumption that processes must be allocated to contiguous blocks of memory.

Paging provides the appearance of a contiguous allocation.

![VM Paging](.gitbook/assets/image%20%2875%29.png)

![](.gitbook/assets/image%20%2846%29.png)

![](.gitbook/assets/image%20%2877%29.png)

![](.gitbook/assets/image%20%289%29.png)

### Page Faults

* What if the page we want is not in memory?
  * Page table entry indicates that the page is not in memory
  * Causes a page fault \(basically, like a cache miss\)
* How do we handle a page fault?
  * OS is responsible for loading a page from disk
  * Process stops until the data is brought into memory
  * Page replacement policy is up to the OS

### Protection and Sharing

* Processes co-exist in memory
* Processes should not access other processes' memory
  * Must protect process address spaces
  * Implies the need for access rights for pages \(like file permissions - R/W/X\)
* Privileged OS data shouldn't be accessible to users
* In some cases, sharing may be desirable
  * Need to control what sharing is allowed

### VM Enforces Protection

* Address spaces are per-process, completely isolated
* Access rights are kept in the page tables
  * Hardware enforces protection, OS is called when violation happens
  * Page tables are in protected OS memory \(only OS can modify them\)
* Avoid leaked information from deallocated pages

![](.gitbook/assets/image%20%2828%29.png)

### Summary

Virtual memory allows mechanisms for:

* Efficient use of physical memory
* Transparent use of physical memory
* Protection and sharing of physical data between processes

