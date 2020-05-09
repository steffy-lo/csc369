# Crash Consistency: FSCK & Journaling

Crash-Consistency Problem: updating persistent data structures in the presence of a power loss or system crash.

Crash Recovery

* When the file system comes back up, run a program to scan the file system structure and restore consistency
* Can't detect if data block didn't get written
* Only verifies if metadata is consistent

### FSCK

* All data blocks pointed to by inodes \(and indirect blocks\) must be marked allocated in the data bitmap 
* All allocated inodes must be in some directory entry 
* Inode link count must match directory entries

1. Superblock: sanity checks
2. Free Blocks:
3. Inode State:
4. Inode Links:
5. Duplicates:
6. Bad Blocks:
7. Directory Checks:

### Journaling \(Write-Ahead Logging\)

Basic idea:

* Write a log on disk of the operation you are about to do, before making changes

If a crash takes place during the actual write =&gt; go back to journal and retry the actual writes

* Don’t need to scan the entire disk, we know what to do!
* Can recover data as well

If a crash happens before journal write finishes, then it doesn’t matter since the actual write has NOT happened at all, so nothing is inconsistent

![Linux Ext3 File System](../.gitbook/assets/image%20%2814%29.png)

What goes in the "log"?

Transaction Structure:

* Starts with a "transaction begin" \(TxBegin\) block, containing a transaction TID
* Followed by blocks with the content to be written
  * Physical Logging: log exact physical content
  * Logical Logging: log more compact logical representation \(e.g., “this update wishes to append data block Db to ﬁle X”, which is a little more complex but can save space in the log and perhaps improve performance\)
* Ends with a "transaction end" \(TxEnd\) block, containing the corresponding TID

![A Journal Entry](../.gitbook/assets/image%20%2852%29.png)

**Checkpointing**: once the transaction is safely on disk, we are ready to overwrite the old structures in the file system

![Data Journaling Example](../.gitbook/assets/image%20%2849%29.png)

Sequence of Operations:

1. Write the transaction \(containing I\[v2\], B\[v2\], Db\) to the log
2. Write the blocks \(I\[v2\], B\[v2\], Db\) to the file system
3. Mark the transaction free in the journal

If there's a crash on the 2nd operation, then we simple redo the transaction, and if there's a crash on the 3rd operation, it's not really a problem since there's nothing to fix.

The tricky part is when it crashes during the 1st operation \(i.e., writing the to the log\)

* The solution to writing each block at a time is too slow!
* Ideally, we want to be able to write to multiple blocks at once
  * Problem: internal disk scheduling writes TxBegin, I\[v2\], B\[v2\], TxEnd, and finally Db and disk may lose power before Db is written --&gt; Looks like a valid transaction!

To avoid this problem, split into two steps:

1. Journal Write Step: Write all except TxEnd to journal
2. Journal Commit Step: Write TxEnd \(only once the write step is completed\)
3. Checkpoint Step: Now that journal entry is safe, write the actual data and metadata to their right locations on the file system
4. Free Step: Mark transaction as free in journal \(circular log\)
   * After the checkpoint step, the transaction is not needed anymore because metadata and data made it safely to disk, so the space can be freed

**Journaling Recovery**

* If crash happens before the transaction is committed to the journal
  * Just skip the pending update
* If crash happens during the checkpoint step \(redo logging\)
  * After reboot, scan the journal and look for committed transactions
  * Replay the transactions
  * After replay, the file system is guaranteed to be consistent

**Metadata Journaling**

* Recovery is much faster with journaling
* However, normal operations are slower
  * Every update, must write to journal first, then do the update \(writing time is at least doubled\)
  * Journal writing may break sequential writing
    * Jump back-and-forth between writes to journal and writes to main region
  * Metadata journaling is similar, except we only write file system metadata \(no actual data\) to the journal

![Metadata Journal](../.gitbook/assets/image%20%2819%29.png)

Important Adjustment: write data BEFORE writing metadata to journal!

1. Write data, wait until it completes
2. Metadata journal write
3. Metadata journal commit
4. Checkpoint metadata
5. Free

Summary

* Journaling ensures file system consistency
* Complexity is in the size of the journal, not the size of disk!
* Metadata journaling is the most commonly used
  * Reduces the amount of traffic to the journal, and provides reasonable consistency guarantees at the same time

