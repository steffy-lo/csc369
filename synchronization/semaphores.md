# Semaphores

* Semaphores are abstract data types that provide synchronization
* They include:
  * An integer counter variable, accessed only through 2 atomic operations
  * The atomic operation wait \(also called P or decrement\)
    * Block while semaphore &lt; 0 and then decrement
  * The atomic operation signal \(also called V or increment\)
    * Increment the variable, unblock a waiting thread if there are any

![Definition: Semaphore](../.gitbook/assets/image%20%2857%29.png)

### Types of Semaphores

* Mutex \(or Binary\) Semaphore \(count = 0/1\)
  * Single access to a resource
  * Mutual exclusion to a critical section
* Counting Semaphore
  * A resource with many units available, or a resource that allows certain kinds of unsynchronized concurrent access \(e.g., reading\)
  * Multiple threads can pass the semaphore
  * Max number of threads is determined by semaphore's initial value, count
    * Mutex has count = 1, counting has count = N

![](../.gitbook/assets/image%20%2815%29.png)

**Atomicity of Wait\( \) and signal\( \)**

* We must ensure that two threads cannot execute wait and signal at the same time
* This is another critical section problem
  * Use lower-level primitives to implement wait and signal
  * Uniprocessor: disable interrupts or use hardware instructions
  * Multiprocessor: use hardware instructions

![A Rendezvous Problem](../.gitbook/assets/image%20%2837%29.png)

It is guaranteed that:

* a1 will happen before b2
* b1 will happen before a2

![](../.gitbook/assets/image%20%2840%29.png)

