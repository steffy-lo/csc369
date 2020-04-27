# Condition Variables \(CVs\)

![](../.gitbook/assets/image%20%288%29.png)

Usage

* Always used together with locks
* The lock protects the shared data that is modified and tested when deciding whether to wait or signal/broadcast

```text
lock_acquire(lock);
while (condition not true) {
    cv_wait(cond, lock);
}
... // do stuff
cv_signal(cond); // or cv_broadcast(cond)
lock_release(lock);
```

Signal vs. Broadcast

* Signal: the OS scheduler will pick one "lucky" thread to wake up
* Broadcast: all the threads wake up, but only one of them re-acquires the mutex \(when the broadcaster releases it\) and can thus go through
  * Every other woken up thread will wait
  * However, the woken up threads are no longer blocked on pthread\_cond\_wait \(no longer in the CV queue\), they wait to acquire the mutex!
  * The condition that made the threads block on pthread\_cond\_wait, might not hold when they get to acquire the mutex

