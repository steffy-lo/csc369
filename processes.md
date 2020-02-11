# Processes

A process contains all of the state for a program in execution and is named using its process id \(pid\)

* An address space
* The code and data for executing the program
* An execution stack encapsulating the state of the procedure calls
* The program counter \(PC\) indicating the next instruction
* A set of general-purpose registers with current values
* A set of operating system resources \(e.g., open files, network connections, signals, etc.\)

![](.gitbook/assets/image%20%281%29.png)

**Keeping Track of Processes**

* OS maintains a collection of state queues that represent the state of all processes in the system
* Typically one queue for each state \(ready, waiting for event X\)
* As a process changes state, its PCB \(process control block\) is unlinked from one queue and linked into another

**From Program to Process**

![](.gitbook/assets/image%20%282%29.png)

1. Create a new process \(i.e., fork\(\) \)
   * Create new PCB, user address space structure
   * Allocate memory
2. Load Executable
   * Initialize start state for process
   * Change state to 'ready'
3. Dispatch process
   * Change state to 'running'

**State Change: Ready to Running**

* Context switch == switch the CPU to another process by
  * Saving the state of the old process
  * Loading the saved state for the new process
* When can this happen?
  * Process calls yield\( \) system call; this is done voluntarily
  * Process makes other system call and is blocked
  * Timer interrupt handler decides to switch processes

![Context Switch](.gitbook/assets/image%20%2839%29.png)

**Process Creation**

* A process is created by another process; the init process \(pid 1\) creates the first process
* In some systems, the parent defines \(or donates\) resources and privileges for its children
  * Unix: process user id is inherited - children of your shell execute with your privileges
* After creating a child, the parent may either wait for it to finish its task or continue in parallel \(or both\)

fork\( \)

* Creates and initializes a new PCB
* Creates a new address space
* Initializes the address space with a copy of the entire contents of the address space of the parent
* Initializes the kernel resources to point to the resources used by parent \(e.g., open files\)
* Places the PCB on the ready queue
* fork\( \) returns in two processes - returns child's pid to parent and 0 to the child

exec\( \)

* Stops the current process
* Loads program "prog" into the process' address space
* Initializes hardware context and args for the new program
* Places the PCB onto the ready queue
* Note: doesn't create a new process

Process Destruction

exit\( \)

* On exit\( \), a process voluntarily releases all resources
* BUT OS can't discard everything immediately
  * Must stop running the process to free everything
  * Requires context switch to another process
  * Parent may be waiting or asking for the return value
* Doesn't cause all data to be freed!
  * Some OS data structures retain the process' exit state
  * It is a zombie until its parent cleans it up

System Calls - "A function call that invokes the operating system"

System Call Dispatch

1. Kernel assigns each system call type a system call number
2. Kernel initializes system call table, mapping each system call number to a function implementing that system call
3. User process sets up system call number and arguments
4. User process runs int N \(on Linux, N = 0x80\)
5. Hardware switches to kernel mode and invoke's kernel's interrupt handler for X \(interrupt dispatch\)
6. Kernel looks up syscall table using system call number
7. Kernel invokes corresponding function
8. Kernel returns by running iret \(interrupt return\)

![Linux Write System Call](.gitbook/assets/image%20%2825%29.png)

