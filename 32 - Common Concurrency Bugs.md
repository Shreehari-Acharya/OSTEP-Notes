## Non-Deadlock Bugs

### Atomicity Violation Bugs

- Definition: A sequence of instructions that should have been executed together (atomically) was not.
- Example: In MySQL, a thread checks if `thd->proc_info` is non-NULL and then uses it in `fputs()`. If interrupted by another thread setting `thd->proc_info` to NULL, it can cause a crash.
- Fix: Use locks to ensure atomic execution of related instructions.

### Order Violation Bugs

- Definition: The desired order between two threads' operations is not enforced.
- Example: In Mozilla, `mThread->State` is accessed before `mThread` is initialized.
- Fix: Use condition variables or semaphores to enforce ordering between threads.

### Summary

- 97% of non-deadlock bugs are either atomicity or order violations.
- Careful thinking and automated tools can help avoid these bugs.

## Deadlock Bugs

### Definition

- Deadlock occurs when threads are stuck waiting for each other to release resources (like locks).
- Example: Thread 1 holds lock L1 and waits for L2, while Thread 2 holds L2 and waits for L1.

### Conditions for Deadlock

1. Mutual exclusion: Threads claim exclusive control of resources.
2. Hold-and-wait: Threads hold resources while waiting for additional ones.
3. No preemption: Resources cannot be forcibly removed from threads.
4. Circular wait: A circular chain of threads, each holding resources requested by the next.

### Prevention Strategies

1. Prevent Circular Wait
    - Enforce a total or partial ordering on lock acquisition.
    - Example: Always acquire L1 before L2.
2. Avoid Hold-and-wait
    - Acquire all locks atomically.
    - Issues: Reduces concurrency, requires knowledge of all needed locks upfront.
3. Allow Preemption
    - Use `pthread_mutex_trylock()` to avoid blocking indefinitely.
    - Issues: Can lead to livelock, complicates code with resource management.
4. Avoid Mutual Exclusion
    - Use lock-free or wait-free data structures.
    - Example: Use compare-and-swap (CAS) for atomic operations.
    - Issues: Complex to implement, limited applicability.

### Avoidance via Scheduling

- Use global knowledge of lock needs to schedule threads safely.
- Example: Dijkstra's Banker's Algorithm.
- Issues: Limited applicability, reduces concurrency.

### Detect and Recover

- Allow occasional deadlocks, then detect and recover.
- Example: Database systems running deadlock detectors.
- Pros: Pragmatic if deadlocks are rare.

## Summary

- Non-deadlock bugs (atomicity and order violations) are common but often easier to fix.
- For deadlocks, prevention (especially lock ordering) is best in practice.
- Wait-free approaches show promise but are complex.
- Consider new programming models (like MapReduce) that avoid locks altogether.

## Questions & Answers:

1. **First let’s make sure you understand how the programs generally work, and some of the key options. Study the code in vector-deadlock.c, as well as in main-common.c and related files.
Now, run ./vector-deadlock -n 2 -l 1 -v, which instantiates two threads (-n 2), each of which does one vector add (-l 1), and does so in verbose mode (-v). Make sure you understand the output. How does the output change from run to run?**
    
    The output is different when we execute it several times. 
    
    some of the output are:
    
    ```bash
    User@Linux:~/ostep-homework/threads-bugs$ ./vector-deadlock -n 2 -l 1 -v
                  ->add(0, 1)
                  <-add(0, 1)
    ->add(0, 1)
    <-add(0, 1)
    User@Linux:~/ostep-homework/threads-bugs$ ./vector-deadlock -n 2 -l 1 -v
    ->add(0, 1)
    <-add(0, 1)
                  ->add(0, 1)
                  <-add(0, 1)
    
    ```
    
2. **Now add the `-d` flag, and change the number of loops (-l) from 1 to higher numbers. What happens? Does the code (always) deadlock?**
    
    the `-d` flag makes it prone to deadlocks. No the code will not work (always) as there is a high possibility of deadlocks as the number of loops is higher.
    
3. **How does changing the number of threads (-n) change the outcome of the program? Are there any values of -n that ensure no deadlock occurs?**
    
    Increasing the number of threads will make the program to cause a deadlock quite often. Yes, there is one such value which will ensure no deadlock, that is 1. Since the number of threads in one, there is no concurrency and deadlocks can never happen.
    
4. **Now examine the code in vector-global-order.c. First, make sure you understand what the code is trying to do; do you understand why the code avoids deadlock? Also, why is there a special case in this vector add() routine when the source and destination vectors are the same?**
    
    The code is assigning deadlocks based on the memory address. The smaller memory address will be locked first.This will prevent deadlocks as they will always acquire the locks in a particular manner every time.
    
    When the source and destination are same, we use only one lock as both the vectors are same. having two locks for a single vector can cause trouble.
    
5. **Now run the code with the following flags: -t -n 2 -l 100000 -d. How long does the code take to complete? How does the total time change when you increase the number of loops, or the number of threads?**
    
    ```bash
    User@Linux:~/ostep-homework/threads-bugs$ ./vector-global-order -t -n 2 -l 100000 -d
    Time: 0.04 seconds
    User@Linux:~/ostep-homework/threads-bugs$ ./vector-global-order -t -n 8 -l 100000 -d
    Time: 0.31 seconds
    User@Linux:~/ostep-homework/threads-bugs$ ./vector-global-order -t -n 8 -l 10000000 -d
    Time: 34.47 seconds
    
    ```
    
    we can clearly see that as we increase the number of threads or loops the time taken to complete also increases.
    
6. **What happens if you turn on the parallelism flag (-p)? How much would you expect performance to change when each thread is working on adding different vectors (which is what -p enables) versus working on the same ones?**
    
    ```bash
    User@Linux:~/ostep-homework/threads-bugs$ ./vector-global-order -t -n 8 -l 10000000 -d -p
    Time: 2.53 seconds
    
    ```
    
    Adding parallelism will reduce the time taken by a program significantly. This is because different threads are working on different vectors and they will not waste time in waiting for some lock to get unlocked. 
    
7. **Now let’s study vector-try-wait.c. First make sure you understand the code. Is the first call to `pthread mutex trylock()` really needed? Now run the code. How fast does it run compared to the global order approach? How does the number of retries, as counted by the code, change as the number of threads increases?**
    
    The first call to `pthread_mutex_trylock()` on `v_dst->lock` is indeed needed. If it weren't there and the lock was already held, the thread would proceed to try to lock `v_src->lock`. If it got that lock but then found `v_dst->lock` was still held when it went back to the top, it would be holding `v_src->lock` while waiting, potentially leading to deadlock.
    
    ```bash
    User@Linux:~/ostep-homework/threads-bugs$ ./vector-global-order -t -n 4 -l 1000000 -d
    Time: 1.49 seconds
    User@Linux:~/ostep-homework/threads-bugs$ ./vector-try-wait -t -n 4 -l 1000000 -d
    Retries: 18384603
    Time: 5.85 seconds
    ```
    
    We can see that it runs slower than the global-order because when it cannot hold on the second lock, it will go back and release the first lock too.
    
8. **Now let’s look at vector-avoid-hold-and-wait.c. What is the main problem with this approach? How does its performance compare to the other versions, when running both with -p and without it?**
    
    This code avoids hold and lock. It does this by acquiring all needed locks at once, using a global lock to make this acquisition atomic.By using a global lock, it essentially serializes all vector additions. Even if two threads want to operate on entirely different vectors, one thread must wait for the other to finish because they both need to acquire the global lock first.
    
9. **Finally, let’s look at vector-nolock.c. This version doesn’t use locks at all; does it provide the exact same semantics as the other versions? Why or why not?**
    
    It can provide same semantics only when we use it with `-p` flag, which will cause it to run parallel on different vectors. If run on same vectors, it will cause a lot of problems as one thread might read a value from `v_src[i]`, then another thread could update both `v_src[i]` and `v_dst[i]` before the first thread writes its result. This leads to lost updates.
    
10. **Now compare its performance to the other versions, both when threads are working on the same two vectors (no -p) and when each thread is working on separate vectors (-p). How does this no-lock version perform?**
    
    The no-lock version will perform excellently when run parallel with `-p` flag. As for the other case it will run fast, but it cannot give the correct results as there is a high chance of lost updates.
