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

![](../.gitbook/assets/image%20%286%29.png)

Now suppose you share this account with someone and the balance is $1000. You each go to separate ATM machines to withdraw - you withdraw $100 and the other person deposits $100.

The problem is that the execution of the two processes can be interleaved. 

![](../.gitbook/assets/image.png)

![](../.gitbook/assets/image%20%2850%29.png)

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

![](../.gitbook/assets/image%20%2835%29.png)

Performance

* The overhead of entering and exiting the CS is small with respect to the work being done within it

### **2-Thread Solutions**

![Solution A](../.gitbook/assets/image%20%2832%29.png)

* **Mutual Exclusion**
  * Yes, turn can only have one value, so if thread A is repeatedly executing the while loop condition then turn is 1 and thread B could pass through the loop. Since turn is only set to 0 when B is leaving the critical section, A cannot enter the critical section until B has left
* **Progress** \(threads waiting for access to CS are not prevented from entering by threads not in CS\)
  * If thread B runs first, it can access the critical section and it will set turn to 0, but suppose thread B immediately wants to get into the critical section again and thread A is still busy running code unrelated to the critical section. Thread B cannot enter into the critical section a second time until thread A goes through the critical section and sets turn to 1. So, progress is not satisfied.
* **No Starvation**
  * Since progress is not satisfied, there's no point in talking about starvation

![Solution B](../.gitbook/assets/image%20%2839%29.png)

* **Mutual Exclusion**
  * Not satisﬁed because if both threads can pass through the while loop before either of them set their own ﬂag to true. No we have both threads in the critical section at the same time.
* There is no point talking about progress or starvation if mutual exclusion is not satisﬁed

**Solution C**

* Combine key ideas of first two attempts for a correct solution
* The threads share the variables turn and flag \(where flag is an array, as before\)
* Basic Idea \(Basis of the **Dekker's Algorithm and Peterson's Algorithm**\):
  * Set own flag \(indicate interest\) and set turn to self
  * Spin waiting while turn is self AND other has flag set \(is interested\)
  * If both threads try to enter their CS at the same time, turn will be set to both 0 and 1 at roughly the same time. Only one of these assignments will last. The final value of turn decides which of the two threads is allowed to enter its CS first

Another Approach is **Lamport's Bakery Algorithm**

* Upon entering each customer \(thread\) get a number
* The customer with the lowest number is served next
* No guarantee that 2 threads do not get the same number
  * In case of a tie, thread with the lowest id is served first
  * Thread ids are unique and totally ordered

