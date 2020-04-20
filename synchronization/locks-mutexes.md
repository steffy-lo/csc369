# Locks \(Mutexes\)

Synchronization mechanism with 2 operations: acquire\( \) and release\( \)

An object associated with a particular critical section that you need to "own" if you wish to execute in that region

```text
acquire(lock);
    // CRITICAL SECTION
release(lock);
```

Downsides:

* Can cause deadlocks if not careful
* Cannot allow multiple concurrent access to a resource

**POSIX Locks**

* POSIX locks are called mutexes \(since locks provide mutual exclusion\)
* A few calls associated with POSIX mutexes:

  * pthread\_mutex\_init\(mutex, attr\) - initializes a mutex
  * pthread\_mutex\_destroy\(mutex\) - destroy a mutex
  * pthread\_mutex\_lock\(mutex\) - acquire the lock
  * pthread\_mutex\_trylock\(mutex\) - try to acquire the lock
  * pthread\_mutex\_unlock\(mutex\) - release the lock

**Initializing and Destroying POSIX Mutexes**

* POSIX mutexes can be statically or dynamically created
  * Statically using PTHREAD\_MUTEX\_INITIALIZER
    * pthread\_mutex\_t mx = PTHREAD\_MUTEX\_INITIALIZER;
    * will initialize the mutex will default attributes
    * only use for static mutexes; no error checking is performed
  * Dynamically, using the pthread\_mutex\_init call

    * int pthread\_mutex\_init\(pthread\_mutex\_t \*mutex,

                                                     const pthread\_mutexattr\_t \*attr\);

    * mutex: the mutex to be initialized
    * attr: the structure whose contents are used at mutex creation to determine the mutex's attributes
* Destroy using pthread\_mutex\_destroy\(pthread\_mutex\_t \*mutex\);
  * mutex: the mutex to be destroyed
    * make sure it's unlocked! \(destroying a locked mutex leads to undefined behaviour\)
  * Note: destroying only applies to dynamically initialized mutexes

**Acquiring and Releasing POSIX Locks**

* Acquire 
  * int pthread\_mutex\_lock\(pthread\_mutex\_t \*mutex\);
    * mutex: the mutex to lock \(acquire\)
    * If mutex is already locked by another thread, the call will block until the mutex is unlocked
  * int pthread\_mutex\_trylock\(pthread\_mutex\_t \*mutex\);
    * mutex: the mutex to TRY to lock \(acquire\)
    * If mutex is already locked by another thread, the call will return a "busy" error code
* Release
  * int pthread\_mutex\_unlock\(pthread\_mutex\_t \*mutex\);
    * mutex: the mutex to unlock \(release\)

### Banking Example

* Bank account balance maintained in one variable "int balance"
* Transactions: deposit or withdraw some amount from the account \(+/- balance\)
* Unprotected, concurrent access to your balance could create race conditions

| Thread 1: Withdraws 100 | Thread 2: Withdraws 100 |
| :--- | :--- |
| int new\_balance = balance - amount; | int new\_balance = balance - amount; |
| balance = new\_balance; | balance = new\_balance; |

End with \(balance - 100\) instead of \(balance - 200\)

Idea: put a lock around the code that modifies balance so only a single thread access it at any given time

```text
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#define NUM_THREADS 200
int balance=0;
pthread_mutex_t bal_mutex;
int main (int argc, char *argv[]) {
    pthread_t thread[NUM_THREADS];
    pthread_mutex_init(&bal_mutex, NULL);
    for (int t = 0; t < NUM_THREADS; t += 2) {
        int rc = pthread_create(&thread[t], NULL, deposit, (void*)10); 
        if (rc != 0) {
            printf("ERROR: pthread_create() returned %d\n", rc);
            exit(-1);
        }
        rc = pthread_create(&thread[t+1], NULL, withdraw, (void*)10); 
        if (rc != 0) {
            printf("ERROR: pthread_create() returned %d\n", rc);
            exit(-1);
        }
    }
    for (int t = 0; t < NUM_THREADS; t++) {
        void *status = NULL;
        int rc = pthread_join(thread[t], &status);
        if (rc != 0) {
            printf("ERROR; return code from "
                               "pthread_join() is %d\n", rc);
            exit(-1);
        }
    }
    printf("Final Balance is %d.\n", balance);
    return 0;
}
```

![](../.gitbook/assets/image%20%2814%29.png)

