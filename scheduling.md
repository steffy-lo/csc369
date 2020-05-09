# Scheduling

### What is Processor Scheduling?

* The allocation of processors to threads over time
* The key to multi-programming
  * We want to increase CPU utilization and job throughput by overlapping I/O and computation
  * Mechanisms: thread states, thread queues
  * Policies: Given more than one runnable thread, how do we choose which to run next? When do we make this decision?

Starting simple, assume that processes run to completion when they are first scheduled. The goal is to minimize average wait time.

![](.gitbook/assets/image%20%2834%29.png)

![](.gitbook/assets/image%20%2856%29.png)

* Average waiting time under FCFS is often quite long
  * Convoy effect: all other processes wait for the one big process to release the CPU
* Non-preemptive

In practice, we don't know how long a thread will run. Threads don't just use the CPU for a some amount of time and then terminate.

### Thread Life Cycle

* Threads repeatedly alternate between computation and I/O
  * Called CPU bursts and I/O bursts
  * Last CPU bursts ends with a call to terminate the thread \(thread\_exit\(\) or equivalent\)
  * CPU-bound: very long CPU bursts, infrequent I/O bursts
  * I/O-bound: short CPU bursts, frequent \(long\) I/O bursts
* During I/O bursts, CPU is not needed

![Process State Diagram](.gitbook/assets/image%20%2858%29.png)

### Scheduling Goals

* All systems
  * Fairness: each process receives fair share of CPU
  * Avoid starvation
  * Policy Enforcement: usage policies should be met
  * Balance: all parts of the system should be busy
* Batch Systems
  * Throughput: maximize jobs completed per hour
  * Turnaround time: minimize time between submission and completion
  * CPU utilization: keep the CPU busy all the time
* Interactive Systems
  * Response time: minimize time between receiving request and starting to produce output
  * Proportionality - "simple" tasks complete quickly
* Real-Time Systems
  * Meet deadlines
  * Predictability

![Focus: Short-Term Scheduler](.gitbook/assets/image%20%2812%29.png)

**Short-Term Scheduler**

* aka dispatching
* needs to be efficient \(fast context switches, fast queue manipulation\)

What happens on dispatch?

* Save currently running process state
* Select next process from ready queue
* Restore state of next process
  * Restore registers and OS control structures
  * Switch to user mode
  * Set PC to next instruction in this process

### Types of Scheduling

* Non-preemptive scheduling
  * Once the CPU has been allocated to a process, it keeps the CPU until it terminates or blocks
  * Suitable for batch scheduling
  * Common for user-level thread packages
* Preemptive scheduling
  * CPU can be taken from a running process and allocated to another
  * Needed in interactive or real-time systems

### Round Robin \(RR\) Algorithm

* Designed for time-sharing systems
* Pre-emptive
* Ready queue is circular
  * Each process is allowed to run for time quantum _**q**_ before being preempted and put back on queue
* Choice of quantum \(aka time slice\) is critical
  * as _q_ --&gt; Infinity, RR --&gt; FCFS
  * as _q --&gt;_ 0, overhead of switching overwhelms useful time
  * we want _q_ to be large w.r.t. the context switch time

#### Priority Scheduling

* A priority, p, is associated with each thread
* Highest priority job is selected from Ready queue \(can be preemptive or non-preemptive\)
* Enforcing this policy is tricky
  * A low priority task may prevent a high priority task from making progress by holding a resource \(priority inversion\)
  * A low priority task may never get to run \(starvation\)

#### Multi-Level Queue \(MLQ\) Scheduling

* Have multiple ready queues
  * Each runnable process is on only one queue
* Threads are permanently assigned to a queue
* Each queue has its own scheduling algorithm
  * Another level of scheduling decides which queue to choose next
  * Usually priority-based

#### Feedback Scheduling

* Adjust criteria for choosing a particular thread based on past history
  * Can boost priority of threads that have waited a long time \(aging\)
  * Can prefer threads that do not use full quantum
  * Can boost priority following a user-input event
  * Can adjust expected next-CPU-burst
* Combine with MLQ to move threads between queues

### Multi-Level Feedback Queue \(MLFQ\)

* A number of distinct queues, each one assigned a different priority level
* Each queue has multiple ready-to-run jobs
* The scheduler always chooses to run the jobs in the queue with the highest priority
* Job start in the highest priority queue
* FEEDBACK:
  * If a job uses an entire time slice, its priority gets reduced \(gets moved to a lower queue\)
  * If the job gives up the CPU before the time slice is up, it stays at the same priority level

#### Priority Inversion

* With all of these schemes where some processes are prioritized over others, there is the possibility of priority inversion
* Priority inversion happens when a lower priority thread prevents a high priority thread from running
  * E.g., a low priority thread holds a lock and is preempted. A high priority thread that wants the lock cannot make progress until the low priority thread runs again
* Solution: Priority Inheritance
  * Priority of task holding the mutex \(semaphore\) inherits the priority of a higher priority task when the higher priority task requests the semaphore
  * This allows the low priority task to preempt the medium priority task

