# Threads

Interprocess Communication

How do processes communicate?

* signals
* pipes
* sockets
* files

Wouldn't it be nice if cooperating processes could just share memory?

Parallel Programs

* To execute a web server, or any parallel program using fork\(\), we need to:
  * Create several processes that execute in parallel
  * Cause each to map to the same address space to share data
  * They are all part of the same computation
  * Have the OS schedule these processes in parallel \(logically or physically\)
* This situation is very inefficient
  * Space: PCB, page tables, etc.
  * Time: Create data structures, fork and copy addr space, etc.
  * Interprocess communicaiton \(IPC\): extra work is needed to share and communicate across isolated processes

The Big Idea

* Separate the address space from the execution state
* Then multiple "threads of execution" can execute in a single address space

Why?

* Sharing: threads can solve a single problem concurrently and can easily share code, heap, and global variables
* Lighter weight: faster to create and destroy, potentially faster context switch times
* Concurrent programming performance gains: overlapping computation and I/O

What is a Thread?

* A thread is a single control flow through a program
* A program with multiple control flows is "multi-threaded"
  * OS must interact with multiple running programs, so it is naturally multi-threaded

![Multi-Threaded Process Address Space](.gitbook/assets/image%20%2841%29.png)

Kernel-Level Threads

* Modern OSs have taken the execution aspect of a process and separated it out into a thread abstraction --&gt; To make concurrency cheaper
* The OS now manages threads and processes
  * All thread operations are implemented in the kernel
  * The OS schedules all of the threads in the system
* Limitations
  * Makes concurrency much cheaper than processes \(less state to allocate and initialize\)
  * For fine-grained concurrency, suffers from too much overhead
    * Thread operations still require system calls. Ideally, want thread operations to be as fast as a procedure call
    * Kernel-level threads have to be general to support the needs of all programmers, languages, runtimes, etc.

User-Level Threads

* To make threads cheap and fast, they need to be implemented at user level
  * Kernel-level threads are managed by the OS
  * User-level threads are managed entirely by the run-time system \(user-level library\)
* User-level threads are small and fast
  * A thread is simply represented by a PC, registers, stack, and small thread control block \(TCB\)
  * Creating a new thread, switching between threads, and synchronizing threads are done via procedure call \(no kernel involvement\)
  * User-level thread operations are up to 100x faster than kernel threads
* Limitations
  * Invisible and hence not well-integrated with the OS
  * As a result, the OS can make poor decisions
    * Scheduling a process with idle threads 
    * Blocking a process whose thread initiated an I/O, even though the process has other threads that can execute
    * De-scheduling a process with a thread holding a lock
  * Solving this requires communication between the kernel and the user-level thread manager

![](.gitbook/assets/image%20%289%29.png)

POSIX Threads Tutorial: [https://computing.llnl.gov/tutorials/pthreads/\#PthreadsAPI](https://computing.llnl.gov/tutorials/pthreads/#PthreadsAPI)

Summary

* Processes are more heavy-weight than threads and have a higher startup/shutdown cost  \(..not by much in Linux though\)
* Processes are safer and more secure \(each process has its own address space\)
  * a thread crash takes down all other threads
  * a thread’s buffer overrun creates security problem for all 

