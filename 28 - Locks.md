## Locks: The Basic Idea

- To execute a series of instructions atomically, programmers annotate source code with locks, putting them around critical sections.
- A lock is a variable that holds the state of the lock at any instant (available/free or acquired/held by a thread).
- `lock()` tries to acquire the lock; if no other thread holds it, the calling thread acquires it and enters the critical section.
- `unlock()` releases the lock, allowing other waiting threads to acquire it.
- Locks provide some control over scheduling to programmers, guaranteeing that only one thread can be active within the critical section.

## Pthread Locks

- POSIX library uses the term `mutex` (mutual exclusion) for locks.
- Example:

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
Pthread_mutex_lock(&lock);
// critical section
Pthread_mutex_unlock(&lock);

```

## Building a Lock

- Requires hardware support (special instructions) and OS support.

## Evaluating Locks

- Criteria:
    1. Mutual Exclusion: Does it prevent multiple threads from entering the critical section?
    2. Fairness: Do threads get a fair shot at acquiring the lock? Is starvation prevented?
    3. Performance: Time overheads in cases of no contention, contention on a single CPU, and contention on multiple CPUs.

## Controlling Interrupts

- Early solution: Disable interrupts for critical sections (single-processor systems).
- Pros: Simple.
- Cons: Requires trusting arbitrary programs with privileged operations, doesn't work on multiprocessors, can lead to lost interrupts.

## Using Loads/Stores

- Using a single flag variable and accessing it via normal loads and stores is insufficient for building a lock (race conditions).

## Building Working Spin Locks with Test-And-Set

- `test-and-set` is a hardware instruction that atomically sets a value and returns the old value.
- Enables building a simple spin lock.

## Evaluating Spin Locks

- Provides mutual exclusion and is correct.
- Not fair (no guarantee that a waiting thread will acquire the lock).
- Performance:
    - Wasteful on a single CPU under contention (threads spin-wait while lock holder is preempted).
    - Works reasonably well on multiple CPUs (if #threads ≈ #CPUs).

## Compare-And-Swap

- Another hardware primitive: `compare-and-swap(ptr, expected, new)`.
- Atomically updates the value at `ptr` to `new` if the value is `expected`, and returns the original value.
- Can be used to build locks similar to `test-and-set`.

## Load-Linked and Store-Conditional

- Pair of instructions: `load-linked` and `store-conditional`.
- `load-linked` fetches a value from memory.
- `store-conditional` updates the value only if no intervening store occurred.
- Can be used to build locks.

## Fetch-And-Add

- Hardware instruction: `fetch-and-add(ptr)`.
- Atomically increments the value at `ptr` and returns the old value.
- Used to build ticket locks, which ensure progress for all threads.

## Too Much Spinning: What Now?

- Spin locks can be inefficient when a thread gets preempted while holding a lock (other threads waste time spinning).
- Need OS support to avoid spinning.

## A Simple Approach: Just Yield, Baby

- `yield()` system call allows a thread to give up the CPU and let another thread run.
- Works well for two threads on one CPU, but wasteful with many threads contending.
- Doesn't address starvation.

## Using Queues: Sleeping Instead of Spinning

- Use a queue to track waiting threads and give control over which thread acquires the lock next.
- Requires OS support like `park()` and `unpark()` (Solaris) or `futex` (Linux) to put threads to sleep and wake them up.

## Different OS, Different Support

- Linux provides `futex` for efficient locks with in-kernel functionality.

## Two-Phase Locks

- Spin for a while hoping to acquire the lock quickly (first phase).
- If unsuccessful, sleep and wait to be woken up when the lock is free (second phase).
- Hybrid approach combining spinning and sleeping.

## Summary

- Real locks are built using hardware primitives and OS support.
- Details and implementations vary across systems (e.g., Solaris, Linux).

## Questions & Answers

1. **Examine flag.s. This code “implements” locking with a single memory flag. Can you understand the assembly?**<br>
It is a simple assembly program which implements a simple spin lock mechanism.
2. **When you run with the defaults, does flag.s work? Use the -M and -R flags to trace variables and registers (and turn on -c to see their values).Can you predict what value will end up in flag?**<br>
When we run it with defaults, flag.s will work fine. Since the default number of threads is set to 2, the program will run for 2 times and update the value ax to 2.

```
User@Linux:~/ostep-homework/threads-locks$ python3 ./x86.py -p flag.s -M flag -R ax,bx -c
ARG seed 0
ARG numthreads 2
ARG program flag.s
ARG interrupt frequency 50
ARG interrupt randomness False
ARG procsched
ARG argv
ARG load address 1000
ARG memsize 128
ARG memtrace flag
ARG regtrace ax,bx
ARG cctrace False
ARG printstats False
ARG verbose False

 flag      ax    bx          Thread 0                Thread 1

    0       0     0
    0       0     0   1000 mov  flag, %ax
    0       0     0   1001 test $0, %ax
    0       0     0   1002 jne  .acquire
    1       0     0   1003 mov  $1, flag
    1       0     0   1004 mov  count, %ax
    1       1     0   1005 add  $1, %ax
    1       1     0   1006 mov  %ax, count
    0       1     0   1007 mov  $0, flag
    0       1    -1   1008 sub  $1, %bx
    0       1    -1   1009 test $0, %bx
    0       1    -1   1010 jgt .top
    0       1    -1   1011 halt
    0       0     0   ----- Halt;Switch -----  ----- Halt;Switch -----
    0       0     0                            1000 mov  flag, %ax
    0       0     0                            1001 test $0, %ax
    0       0     0                            1002 jne  .acquire
    1       0     0                            1003 mov  $1, flag
    1       1     0                            1004 mov  count, %ax
    1       2     0                            1005 add  $1, %ax
    1       2     0                            1006 mov  %ax, count
    0       2     0                            1007 mov  $0, flag
    0       2    -1                            1008 sub  $1, %bx
    0       2    -1                            1009 test $0, %bx
    0       2    -1                            1010 jgt .top
    0       2    -1                            1011 halt

```

3. **Change the value of the register %bx with the -a flag (e.g., -a bx=2,bx=2 if you are running just two threads). What does the code do? How does it change your answer for the question above?**<br>
Since we set bx to value 2, each thread will loop for 2 times.Therefore the value of ax will be 2+2 = 4.

```
shreehari@Ubuntu:~/ostep-homework/threads-locks$ python3 ./x86.py -p flag.s -M flag -R ax,bx -a bx=2,bx=2 -c
ARG seed 0
ARG numthreads 2
ARG program flag.s
ARG interrupt frequency 50
ARG interrupt randomness False
ARG procsched
ARG argv bx=2,bx=2
ARG load address 1000
ARG memsize 128
ARG memtrace flag
ARG regtrace ax,bx
ARG cctrace False
ARG printstats False
ARG verbose False

 flag      ax    bx          Thread 0                Thread 1

    0       0     2
    0       0     2   1000 mov  flag, %ax
    0       0     2   1001 test $0, %ax
    0       0     2   1002 jne  .acquire
    1       0     2   1003 mov  $1, flag
    1       0     2   1004 mov  count, %ax
    1       1     2   1005 add  $1, %ax
    1       1     2   1006 mov  %ax, count
    0       1     2   1007 mov  $0, flag
    0       1     1   1008 sub  $1, %bx
    0       1     1   1009 test $0, %bx
    0       1     1   1010 jgt .top
    0       0     1   1000 mov  flag, %ax
    0       0     1   1001 test $0, %ax
    0       0     1   1002 jne  .acquire
    1       0     1   1003 mov  $1, flag
    1       1     1   1004 mov  count, %ax
    1       2     1   1005 add  $1, %ax
    1       2     1   1006 mov  %ax, count
    0       2     1   1007 mov  $0, flag
    0       2     0   1008 sub  $1, %bx
    0       2     0   1009 test $0, %bx
    0       2     0   1010 jgt .top
    0       2     0   1011 halt
    0       0     2   ----- Halt;Switch -----  ----- Halt;Switch -----
    0       0     2                            1000 mov  flag, %ax
    0       0     2                            1001 test $0, %ax
    0       0     2                            1002 jne  .acquire
    1       0     2                            1003 mov  $1, flag
    1       2     2                            1004 mov  count, %ax
    1       3     2                            1005 add  $1, %ax
    1       3     2                            1006 mov  %ax, count
    0       3     2                            1007 mov  $0, flag
    0       3     1                            1008 sub  $1, %bx
    0       3     1                            1009 test $0, %bx
    0       3     1                            1010 jgt .top
    0       0     1                            1000 mov  flag, %ax
    0       0     1                            1001 test $0, %ax
    0       0     1                            1002 jne  .acquire
    1       0     1                            1003 mov  $1, flag
    1       3     1                            1004 mov  count, %ax
    1       4     1                            1005 add  $1, %ax
    1       4     1                            1006 mov  %ax, count
    0       4     1                            1007 mov  $0, flag
    0       4     0                            1008 sub  $1, %bx
    0       4     0                            1009 test $0, %bx
    0       4     0                            1010 jgt .top
    0       4     0                            1011 halt

```

4. **Set bx to a high value for each thread, and then use the -i flag to generate different interrupt frequencies; what values lead to a bad outcomes? Which lead to good outcomes?** <br>
I set both bx values to 50.The expected output should be 100. The assembly instruction has a total of 11 instructions. if we set -i flag to a value that is a multiple of 11, we get our expected results. any other value will give random outputs.
5. **Now let’s look at the program test-and-set.s. First, try to understand the code, which uses the xchg instruction to build a simple locking primitive. How is the lock acquire written? How about lock release?** <br>
The lock acquisition is implemented using the xchg instruction. The code tries to swap the value 1 with the mutex variable atomically. If the mutex was initially 0, it becomes 1, and the code gets 0 back, indicating that the lock was successfully acquired. If the mutex was non-zero, it means the lock is already taken, and the code enters a loop to try again later.
The lock is released by simply setting the mutex variable to 0 using the mov $0, mutex instruction.
6. **Now run the code, changing the value of the interrupt interval (-i) again, and making sure to loop for a number of times. Does the code always work as expected? Does it sometimes lead to an inefficient use of the CPU? How could you quantify that?** <br>
The code will always work as expected.
the interrupt interval can impact the efficiency of CPU utilization. If the interrupt interval is set to a high value, it means that processes will run for longer periods without being interrupted, potentially leading to inefficient use of the CPU
7. **Use the -P flag to generate specific tests of the locking code. For example, run a schedule that grabs the lock in the first thread, but then tries to acquire it in the second. Does the right thing happen? What else should you test?** <br>
Yes the code works as expected.
8. **Now let’s look at the code in peterson.s, which implements Peterson’s algorithm (mentioned in a sidebar in the text). Study the code and see if you can make sense of it.** <br>
9. **Now run the code with different values of -i. What kinds of different behavior do you see? Make sure to set the thread IDs appropriately (using -a bx=0,bx=1 for example) as the code assumes it.** <br>
The code works as expected and it runs faster. There is no wastage of CPU cycles.
10. **Can you control the scheduling (with the -P flag) to “prove” that the code works? What are the different cases you should show hold? Think about mutual exclusion and deadlock avoidance.** <br>
`User@Linux:~/ostep-homework/threads-locks$ python3 ./x86.py -p peterson.s -a bx=0,bx=1 -c -R ax,bx -i 3 -P 1100`
11. **Now study the code for the ticket lock in ticket.s. Does it match the code in the chapter? Then run with the following flags: -a bx=1000,bx=1000 (causing each thread to loop through the critical section 1000 times). Watch what happens; do the threads spend much time spin-waiting for the lock?** <br>
Yes, it matches the code in the chapter.Yes it does spend time spin-waiting for lock
12. **How does the code behave as you add more threads?**
The code behaves more slower as we add more threads.
13. **Now examine yield.s, in which a yield instruction enables one thread to yield control of the CPU (realistically, this would be an OS primitive, but for the simplicity, we assume an instruction does the task). Find a scenario where test-and-set.s wastes cycles spinning, but yield.s does not. How many instructions are saved? In what scenarios do these savings arise?** <br>
When we set -i to a less value, test-and-set.c wastes cycles spinning, but yield.s does not.
14. **Finally, examine test-and-test-and-set.s. What does this lock do? What kind of savings does it introduce as compared to test-and-set.s?**
It spins for a little time and then goes to sleep, also known as two phase locks.
