---
description: >-
  Redundant Array of Inexpensive Disks: a technique to use multiple disks in
  concert to build a faster, bigger, and more reliable disk system
---

# RAID

Redundancy: have more than one copy of data to prevent data loss!

Redundancy Strategies:

* Data Duplicated - mirror images, redundant full copy
  * If one disk fails, we have the mirror
* Data spread out across multiple disks with redundancy
  * Can recover from a disk failure by reconstructing data

Concepts:

* Redundancy/Mirroring: keep multiple copies of the same block on different drives, in case drive fails
* Parity information: XOR each bit from 2 drives, store checksum on 3rd drive

### RAID Level 0: Striping

* No redundancy
* Serves as an upper bound on performance and capacity
* Files are divided across disks
* Improves throughput
* If one drive fails, the whole volume is lost

Chunk size mostly affects performance of the array

* A small chunk size implies that many ﬁles will get striped across many disks, thus increasing the parallelism of reads and writes to a single ﬁle; however, the positioning time to access blocks across multiple disks increases, because the positioning time for the entire request is determined by the maximum of the positioning times of the requests across all drives
* A big chunk size, on the other hand, reduces such intra-ﬁle parallelism, and thus relies on multiple concurrent requests to achieve high throughput. However, large chunk sizes reduce positioning time; if, for example, a single ﬁle ﬁts within a chunk and thus is placed on a single disk, the positioning time incurred while accessing it will just be the positioning time of a single disk

### RAID Level 1: Mirroring

* Capacity is half
* Any drive can serve a read
* Write throughput is slower
* If one drive fails, no data lost

![Standard RAID Levels](../.gitbook/assets/image%20%2822%29.png)

### RAID Level 4: Saving Space With Parity

* Attempts to reduce the huge space penalty paid by mirrored systems using parity at a cost for performance
* Invariant: the number of 1s in any row, including the parity bit, must be an even \(not odd\) number
* Recovery involves reconstruction of the right value from parity bits
* Can only tolerate 1 disk failure; if more than one disk is lost, cannot reconstruct
* Suffers the **small-write** problem: parity disk bottleneck that has been overcome in RAID Level 5

### RAID Level 5: Rotating Parity

![](../.gitbook/assets/image%20%2844%29.png)

### Comparison

![](../.gitbook/assets/image%20%2823%29.png)

