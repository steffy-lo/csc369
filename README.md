# Introduction to OS

### What is the Operating System?

![](.gitbook/assets/image.png)

The Operating System \(OS\) is in charge of making sure the system operates **correctly** and **efﬁciently** in an **easy-to-use** manner.

How does it relate to the other parts of a computer system?

* Convenient abstraction of hardware and software
* Protection, Security, Authentication
* Communication

Goals of OS

1. Primary: convenience for the user
   * It must be easier to compute with the OS than without it
2. Secondary: efficient operation of the computer system

Note: the two goals are often contradictory and hence, which goals take precedence depends on the purpose of the computer system

Roles of OS

* Virtual Machine
  * Extends and simplifies interface to physical machine
  * Provides a library functions accessible through an API
* Resource Allocator
  * Allows the proper use of resources \(hardware, software, data\) in the operation of the computer system
  * provides an environment in within which other programs can do useful work
* Control Program
  * Controls the execution of user programs to prevent errors and improper use of the computer
  * Especially concerned with the operation and control of I/O devices

Major OS Themes

* Virtualization
  * Present physical resource as a more general, powerful, or easy-to-use form of itself
  * Present illusion of multiple \(or unlimited\) resources where only one \(or a few\) really exist

    Examples: CPU, Memory
* Concurrency
  * Coordinate multiple activities to ensure correctness
* Persistence
  * Some data need to survive crashes and power failures

Need abstractions, mechanisms, and policies for all

