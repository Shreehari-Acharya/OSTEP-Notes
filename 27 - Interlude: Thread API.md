## Thread Creation

- To create a new thread in POSIX, use `pthread_create()`:

```c
int pthread_create(pthread_t *thread,
                   const pthread_attr_t *attr,
                   void *(*start_routine)(void*),
                   void *arg);

```

- `thread`: pointer to `pthread_t` structure to interact with the new thread
- `attr`: thread attributes (e.g., stack size, priority), or NULL for defaults
- `start_routine`: function pointer for the new thread to start running
- `arg`: argument to pass to `start_routine`

## Thread Completion

- To wait for a thread to complete, use `pthread_join()`:

```c
int pthread_join(pthread_t thread, void **value_ptr);

```

- `thread`: specifies which thread to wait for
- `value_ptr`: pointer to receive the return value of the thread

## Locks

- For mutual exclusion in critical sections, use locks:

```c
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);

```

- Locks must be initialized, either statically or dynamically:

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER; // static
int rc = pthread_mutex_init(&lock, NULL); // dynamic

```

- Other lock functions: `pthread_mutex_trylock()`, `pthread_mutex_timedlock()`

## Condition Variables

- For signaling between threads, use condition variables:

```c
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
int pthread_cond_signal(pthread_cond_t *cond);

```

- `pthread_cond_wait()` puts the calling thread to sleep until signaled
- `pthread_cond_signal()` wakes a sleeping thread
- Condition variables should be used with a lock

## Compiling and Running

- Include `pthread.h` header
- Link with `pthread` flag when compiling

## Tips

- Keep thread interactions simple and minimal
- Initialize locks and condition variables
- Check return codes
- Be careful passing arguments to and returning values from threads
- Each thread has its own stack
- Always use condition variables for thread signaling, not flags
- Use manual pages for more details

## Questions & Answers:

1. **First build main-race.c. Examine the code so you can see the (hopefully obvious) data race in the code. Now run helgrind (by typing `valgrind --tool=helgrind main-race`) to see how it reports the race. Does it point to the right lines of code? What other information does it give to you?**
    
yes it point to the right lines of code. It also give information about the number of threads       created, whether they had locks or not etc.
    

```bash
==4323== Possible data race during read of size 4 at 0x10C014 by thread #1
==4323== Locks held: none
==4323==    at 0x109236: main (main-race.c:15)
==4323== 
==4323== This conflicts with a previous write of size 4 by thread #2
==4323== Locks held: none
==4323==    at 0x1091BE: worker (main-race.c:8)
==4323==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==4323==    by 0x4911A93: start_thread (pthread_create.c:447)
==4323==    by 0x499EA33: clone (clone.S:100)
==4323==  Address 0x10c014 is 0 bytes inside data symbol "balance"
```

2. **What happens when you remove one of the offending lines of code?Now add a lock around one of the updates to the shared variable, and then around both. What does helgrind report in each of these cases?**
    
Removing one of the offending lines of code and using Helgrind:
    
We do not get any error
    

```bash
==4575== Helgrind, a thread error detector
==4575== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==4575== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==4575== Command: ./main-race
==4575== 
==4575== 
==4575== Use --history-level=approx or =none to gain increased speed, at
==4575== the cost of reduced accuracy of conflicting-access information
==4575== For lists of detected and suppressed errors, rerun with: -s
==4575== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)

```

Adding lock around one of the updates;

Modified Code:

```c
#include "common_threads.h"

int balance = 0;
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER; // creating a lock variable
void* worker(void* arg) {
    pthread_mutex_lock(&lock); // locking before the critical region
    balance++; // unprotected access
    pthread_mutex_unlock(&lock); //Unlocking after critical region
    return NULL;
}

int main(int argc, char *argv[]) {
    pthread_t p;
    Pthread_create(&p, NULL, worker, NULL);
    balance++; // unprotected access
    Pthread_join(p, NULL);
    return 0;
}

```

Adding lock around one of the updates gives us race-condition.

```bash
==4641==  Lock at 0x10C060 was first observed
==4641==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==4641==    by 0x109207: worker (main-race.c:8)
==4641==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==4641==    by 0x4911A93: start_thread (pthread_create.c:447)
==4641==    by 0x499EA33: clone (clone.S:100)
==4641==  Address 0x10c060 is 0 bytes inside data symbol "lock"
==4641== 
==4641== Possible data race during read of size 4 at 0x10C040 by thread #1
==4641== Locks held: none
==4641==    at 0x109298: main (main-race.c:17)
==4641== 
==4641== This conflicts with a previous write of size 4 by thread #2
==4641== Locks held: 1, at address 0x10C060
==4641==    at 0x109211: worker (main-race.c:9)
==4641==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==4641==    by 0x4911A93: start_thread (pthread_create.c:447)
==4641==    by 0x499EA33: clone (clone.S:100)
==4641==  Address 0x10c040 is 0 bytes inside data symbol "balance"
==4641== 

```

Adding Locks around both the updates on shared variable:

Modified Code:

```c
#include <stdio.h>

#include "common_threads.h"

int balance = 0;
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
void* worker(void* arg) {
    pthread_mutex_lock(&lock);
    balance++; // unprotected access
    pthread_mutex_unlock(&lock); 
    return NULL;
}

int main(int argc, char *argv[]) {
    pthread_t p;
    Pthread_create(&p, NULL, worker, NULL);
    pthread_mutex_lock(&lock);
    balance++; // unprotected access
    pthread_mutex_unlock(&lock);
    Pthread_join(p, NULL);
    return 0;
}
```

We do not get error as we have locked the critical region on both places

```bash
==4707== Helgrind, a thread error detector
==4707== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==4707== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==4707== Command: ./main-race
==4707== 
==4707== 
==4707== Use --history-level=approx or =none to gain increased speed, at
==4707== the cost of reduced accuracy of conflicting-access information
==4707== For lists of detected and suppressed errors, rerun with: -s
==4707== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 7 from 7)

```

3. **Now let’s look at main-deadlock.c. Examine the code. This code has a problem known as deadlock (which we discuss in much more depth in a forthcoming chapter). Can you see what problem it might have?**
     - Thread `p1` (with argument `0`) acquires the lock on `m1`.
     - Thread `p2` (with argument `1`) acquires the lock on `m2`.
     - Now, `p1` is waiting to acquire the lock on `m2`, and `p2` is waiting to acquire the lock on `m1`.
    
    This situation leads to a deadlock, where both threads are stuck waiting for each other to release the lock they need, resulting in an indefinite wait.
    
4. **Now run helgrind on this code. What does helgrind report?**
    
    helgrind does inform about this error
    
    ```bash
    ==5027== 
    ==5027== Thread #3: lock order "0x10C040 before 0x10C080" violated
    ==5027== 
    ==5027== Observed (incorrect) order is: acquisition of lock at 0x10C080
    ==5027==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
    ==5027==    by 0x109288: worker (main-deadlock.c:13)
    ==5027==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
    ==5027==    by 0x4911A93: start_thread (pthread_create.c:447)
    ==5027==    by 0x499EA33: clone (clone.S:100)
    ==5027== 
    ==5027==  followed by a later acquisition of lock at 0x10C040
    ==5027==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
    ==5027==    by 0x1092C3: worker (main-deadlock.c:14)
    ==5027==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
    ==5027==    by 0x4911A93: start_thread (pthread_create.c:447)
    ==5027==    by 0x499EA33: clone (clone.S:100)
    ==5027== 
    
    ```
    

5. **Now run helgrind on main-deadlock-global.c. Examine the code; does it have the same problem that main-deadlock.c has? Should helgrind be reporting the same error? What does this tell you about tools like helgrind?**
    
    No the code does not have the same problem that main-deadlock.c has.
    
    Result from helgrind:
    

```bash
==5154== Thread #3: lock order "0x10C080 before 0x10C0C0" violated
==5154== 
==5154== Observed (incorrect) order is: acquisition of lock at 0x10C0C0
==5154==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==5154==    by 0x1092C3: worker (main-deadlock-global.c:15)
==5154==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==5154==    by 0x4911A93: start_thread (pthread_create.c:447)
==5154==    by 0x499EA33: clone (clone.S:100)
==5154== 
==5154==  followed by a later acquisition of lock at 0x10C080
==5154==    at 0x48512DC: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==5154==    by 0x1092FE: worker (main-deadlock-global.c:16)
==5154==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
==5154==    by 0x4911A93: start_thread (pthread_create.c:447)
==5154==    by 0x499EA33: clone (clone.S:100)
==5154== 

```

The helgrind does tell that there may be a race condition,as its primary focus is on detecting data races, not necessarily deadlocks or other synchronization issues. Even though the code has been modified to prevent deadlocks, there may still be potential data races that Helgrind could report.

6. **Let’s next look at main-signal.c. This code uses a variable (done)to signal that the child is done and that the parent can now continue.Why is this code inefficient? (what does the parent end up spending its time doing, particularly if the child thread takes a long time
to complete?)**
    
    The code is inefficient as the line `while(done == 0)` in the main keeps on running and consumes unnecessary CPU cycles. The parent keeps on checking the done variable with the value 0 until the child thread finishes. 
    
7. **Now run helgrind on this program. What does it report? Is the code correct?**
    
    Results from the helgrind:
    
    ```bash
    ==5324== Possible data race during read of size 4 at 0x10C014 by thread #1
    ==5324== Locks held: none
    ==5324==    at 0x109245: main (main-signal.c:16)
    ==5324== 
    ==5324== This conflicts with a previous write of size 4 by thread #2
    ==5324== Locks held: none
    ==5324==    at 0x1091C8: worker (main-signal.c:9)
    ==5324==    by 0x4854B7A: ??? (in /usr/libexec/valgrind/vgpreload_helgrind-amd64-linux.so)
    ==5324==    by 0x4911A93: start_thread (pthread_create.c:447)
    ==5324==    by 0x499EA33: clone (clone.S:100)
    ==5324==  Address 0x10c014 is 0 bytes inside data symbol "done"
    ==5324== 
    this should print last
    ==5324== 
    
    ```
    

Although the code is correct but inefficient, valgrind reports that there is a race condition as we are using the `done` variable in main as well as in the child thread.

8. **Now look at a slightly modified version of the code, which is found in main-signal-cv.c. This version uses a condition variable to do the signaling (and associated lock). Why is this code preferred to the previous version? Is it correctness, or performance, or both?**
    
    the version with the condition variable and mutex is preferred for both correctness and performance reasons:
    
    1. **Correctness:** It avoids race conditions and ensures proper synchronization between the threads.
    2. **Performance:** It avoids busy-waiting and allows the operating system to efficiently manage CPU resources by blocking the waiting thread instead of wasting cycles in a loop.
9. **Once again run helgrind on main-signal-cv. Does it report any errors?**
    
    No the helgrind does not report any error:
    
    ```bash
    ==5449== Helgrind, a thread error detector
    ==5449== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
    ==5449== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
    ==5449== Command: ./main-signal-cv
    ==5449== 
    this should print first
    this should print last
    ==5449== 
    ==5449== Use --history-level=approx or =none to gain increased speed, at
    ==5449== the cost of reduced accuracy of conflicting-access information
    ==5449== For lists of detected and suppressed errors, rerun with: -s
    ==5449== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 12 from 12)
    
    ```
