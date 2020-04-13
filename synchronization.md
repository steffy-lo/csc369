# Synchronization

Arbitrary interleaving of thread execution can have unexpected consequences

* We need a way to restrict the possible interleavings of executions
* Scheduling is invisible to the application
* Synchronization is the mechanism that gives us this control

**Flavours**

1. Enforce single use of a shared resource
   * Called the critical selection problem
   * E.g. Two threads printing to the console
     * T1: printf\("Hello"\);
     * T2: printf\("Goodbye"\);
     * Result is "HelloGoodbye" OR "GoodbyeHello", but never "HeGooldloye" or some other mixture
2. Control Order of Thread Execution
   * E.g. parent waits for child to finish, ensure menu prints prompt after all output from thread running program

**A Motivating Example**

Suppose we write functions to handle withdrawals and deposits to bank account:

![](.gitbook/assets/image%20%285%29.png)

Now suppose you share this account with someone and the balance is $1000. You each go to separate ATM machines to withdraw - you withdraw $100 and the other person deposits $100.

The problem is that the execution of the two processes can be interleaved. 

![](.gitbook/assets/image.png)

![](.gitbook/assets/image%20%2837%29.png)

**What Went Wrong?**

* Two concurrent threads manipulated a shared resource \(the account\) without any synchronization
  * Outcome depends on the order in which accesses take place \(**race condition**\)
* We need to ensure that only one thread at a time can manipulate the shared resource

**Mutual Exclusion**

* Given:
  * A set of threads: T0, T1, ..., Tn
  * A set of resources shared between threads
  * A segment of code which accesses the share resources, called the critical section \(CS\)
* We want to ensure that:
  * Only one thread at a time can execute in the critical section
  * All other threads are forced to wait on entry
  * When a thread leaves the CS, another can enter

Critical Section Requirements:

1. Mutual Exclusion
   * If one thread is in the CS, then no other is
2. Progress
   * If no thread is in the CS, and some threads want to enter CS, only threads trying to get into the CS section can influence the choice of which thread enters next, and choice cannot be postponed indefinitely
3. Bounded waiting \(no starvation\)
   * If some thread T is waiting on the CS, then there is a limit on the number of times other threads can enter CS before this thread is granted access

Performance

* The overhead of entering and exiting the CS is small with respect to the work being done within it

