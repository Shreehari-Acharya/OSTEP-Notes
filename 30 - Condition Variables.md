## Condition Variables

- Condition variables are a synchronization primitive used in addition to locks to solve certain concurrency problems.
- They allow threads to sleep/wait when some program state is not as desired and wake up when the state changes.

### Definition and Routines

- A condition variable is an explicit queue that threads can put themselves on when some condition is not met.
- Other threads can then wake one or more waiting threads by signaling on the condition variable.
- To declare a condition variable: `pthread_cond_t c;`
- Operations on condition variables:
    - `pthread_cond_wait(pthread_cond_t *c, pthread_mutex_t *m)`: Releases the lock `m` and puts the calling thread to sleep on the condition `c`.
    - `pthread_cond_signal(pthread_cond_t *c)`: Wakes one thread waiting on the condition `c`.

### The Producer/Consumer (Bounded Buffer) Problem

- The producer/consumer problem models multiple producer threads generating data items and placing them in a shared buffer, and multiple consumer threads taking the items from the buffer.
- It requires synchronized access to the shared buffer to prevent race conditions.

### Solution

- Use a shared buffer with `put()` and `get()` functions to add/remove items.
- Producer threads wait on a condition variable `empty` when the buffer is full and signal `fill` after adding an item.
- Consumer threads wait on `fill` when the buffer is empty and signal `empty` after removing an item.
- Use `while` loops (not `if` statements) when checking conditions to handle spurious wakeups and ensure the condition still holds after being signaled.

### Covering Conditions

- In some cases, it's not clear which specific waiting thread(s) should be woken up when a condition changes.
- The solution is to use `pthread_cond_broadcast()` instead of `pthread_cond_signal()` to wake all waiting threads.
- This is called a "covering condition" as it covers all cases where a thread needs to wake up, but may wake too many threads.

### Tips

- Always hold the lock when calling `signal()` or `wait()` on a condition variable.
- Use `while` loops (not `if` statements) when checking conditions to handle spurious wakeups and ensure the condition still holds after being signaled.

## Questions & Answers:

1. **Our first question focuses on `main-two-cvs-while.c` (the working solution). First, study the code. Do you think you have an understanding of what should happen when you run the program?**
    
    The program will produce depending upon the -l flag, and the consumer will consume it, without any race conditions
    
2. **Run with one producer and one consumer, and have the producer produce a few values. Start with a buffer (size 1), and then increase it. How does the behavior of the code change with larger buffers? (or does it?) What would you predict num full to be with different buffer sizes (e.g., `-m 10`) and different numbers of produced items (e.g., `-l 100`), when you change the consumer sleep string from default (no sleep) to `-C 0,0,0,0,0,0,1`?**
    
    With a buffer size of 1 (`-m 1`), the producer can only put one item into the buffer before having to wait for the consumer to take it out.
    
    With a larger buffer size (e.g., `-m 10`), the producer can add up to 10 items into the buffer before having to wait for the consumer. 
    
    With `l 100` (100 produced items) and `m 10` (buffer size 10), `num_full` will fluctuate between 0 and 10, since the producer can fill the entire buffer before waiting for the consumer.
    
    With the consumer sleep string `-C 0,0,0,0,0,0,1`, This means the producer can run for longer stretches, potentially keeping `num_full` near the buffer size most of the time before the consumer wakes up and consumes some items.
    
3. **If possible, run the code on different systems (e.g., a Mac and Linux). Do you see different behavior across these systems?**
    
    
4. **Let’s look at some timings. How long do you think the following execution, with one producer, three consumers, a single-entry shared buffer, and each consumer pausing at point c3 for a second, will take? `./main-two-cvs-while -p 1 -c 3 -m 1 -C 0,0,0,1,0,0,0:0,0,0,1,0,0,0:0,0,0,1,0,0,0 -l 10 -v -t`** 
    
    It took around 12 seconds in my system. This is because we pause right before going to sleep, and during this time the producer will fill the buffer. This will also make the other two consumers useless as they never get the chance to consume.
    
5. **Now change the size of the shared buffer to 3 (`-m 3`). Will this make any difference in the total time?**
    
    Yes, increasing the size of the shared buffer to 3 reduced the time from 12 seconds to 11 seconds. The difference is because the consumer can consume more units at a time due to increased buffer size.
    
6. **Now change the location of the sleep to c6 (this models a consumer taking something off the queue and then doing something with it), again using a single-entry buffer. What time do you predict in this case? `./main-two-cvs-while -p 1 -c 3 -m 1 -C 0,0,0,0,0,0,1:0,0,0,0,0,0,1:0,0,0,0,0,0,1 -l 10 -v -t`** 
    
    In this case, the total time will be shorter because the consumer can immediately consume the item from the buffer without sleeping first. This will allow the producer to run for longer stretches before having to wait for a consumer to finish processing the item.
    
7. **Finally, change the buffer size to 3 again (`-m 3`). What time do you predict now?**
    
    **Little less than before, because the producer can produce more at once.**
    
8. **Now let’s look at `main-one-cv-while.c`. Can you configure a sleep string, assuming a single producer, one consumer, and a buffer of size 1, to cause a problem with this code?**
    
    if we use the sleep string `-C 0,0,0,0,0,1,0` for the consumer, it will sleep for 1 time unit at point `c5`. This means that after consuming an item from the buffer, the consumer will signal the condition variable (`c5`) but won't release the lock (`c6`) immediately.
    
9. **Now change the number of consumers to two. Can you construct sleep strings for the producer and the consumers so as to cause a problem in the code?**
    
    Producer sleep string: -P 0,0,0,0,0,1,0 (sleep for 1 time unit at point p5 after signaling the condition variable but before releasing the lock)
    Consumer 1 sleep string: -C 0,0,0,1,0,0,0 (sleep for 1 time unit at point c3 before checking if the buffer is empty)
    Consumer 2 sleep string: `-C 0,0,0,1,0,0,0` (same as Consumer 1)
    
10. **Now examine `main-two-cvs-if.c`. Can you cause a problem to happen in this code? Again consider the case where there is only one consumer, and then the case where there is more than one.**
    
    With one consumer:
    We can use the sleep string
    
    `-C 0,0,0,1,0,0,0`
    
    for the consumer. This will make the consumer sleep for 1 time unit at point c3 after acquiring the lock and checking if the buffer is empty (c2), but before waiting on the condition variable (c3).
    
    With more than one consumer:
    We can use the sleep strings
    
    `-C 0,0,0,1,0,0,0:0,0,0,1,0,0,0`
    
    This will make both consumers sleep for 1 time unit at point c3 after acquiring the lock and checking if the buffer is empty (c2), but before waiting on the condition variable (c3).
    
11. **Finally, examine `main-two-cvs-while-extra-unlock.c`. What problem arises when you release the lock before doing a put or a get? Can you reliably cause such a problem to happen, given the sleep strings? What bad thing can happen?**
    
    In the `main-two-cvs-while-extra-unlock.c` code, the problem arises when the lock is released before calling `do_fill` (for producers) or `do_get` (for consumers). This violates the mutual exclusion principle, as it allows multiple threads to access the shared buffer concurrently, leading to potential race conditions and data corruption
    
    we can use the sleep string `-P 0,0,0,1,0,0,0` for the producer and `-C 0,0,0,0,1,0,0` for the consumer.
