## 1. Introduction

- Processes provide the illusion of multiple programs running at the same time (virtual CPUs) and private virtual memory (address space abstraction)
- A thread is an abstraction for a single running process - it is a single point of execution with its own program counter (PC) and registers
- Threads within a process share the same address space, allowing data sharing

## 2. Why Use Threads?

- Parallelism: Threads allow utilizing multiple CPUs to speed up tasks like operations on large data structures
- Avoiding Blocking: While one thread waits for I/O, other threads can continue running to overlap useful work with I/O waits

## 3. Thread Creation Example

- Creating threads is like making a function call, but the called routine runs independently as a new thread
- Threads can run in different orders based on scheduling decisions, making reasoning about execution difficult

## 4. Shared Data Issue

- Concurrent access to shared data can lead to race conditions and non-deterministic results
- Example shows two threads incrementing a shared counter, but the interleaved execution causes the counter to be incremented just once instead of twice

## 5. The Race Condition Problem

- Critical section: Code that accesses a shared resource and must not be executed concurrently by multiple threads
- Race condition: Multiple threads enter the critical section at roughly the same time, leading to unexpected results
- Indeterminate program: A program with race conditions that produces different outputs on each run
- Need mutual exclusion to ensure only one thread executes a critical section at a time

## 6. The Wish for Atomicity

- Having powerful hardware instructions that perform multi-instruction sequences atomically could solve race conditions
- But generally not feasible, so we build higher-level synchronization primitives using basic hardware support

## 7. Waiting for Another Issue

- Another concurrency issue is when one thread must wait for another to complete some action before proceeding
- E.g., a thread waiting for disk I/O to complete before continuing

## 8. Why Study Concurrency in OS Class?

- The OS was the first concurrent program, handling interrupts required proper synchronization of kernel data structures
- Application programmers later needed similar support for multi-threaded programs sharing data structures

## 9. Key Concurrency Terms

- **Critical Section**: A piece of code that accesses a shared resource, usually a variable or data structure. It must not be executed concurrently by multiple threads.
- **Race Condition**: A race condition (or data race) arises when multiple threads of execution enter a critical section at roughly the same time. Both threads attempt to update the shared data structure, leading to a surprising (and perhaps undesirable) outcome.
- **Indeterminate Program**: A program that consists of one or more race conditions. The output of the program varies from run to run, depending on which threads ran when and how their execution was interleaved. The outcome is thus not deterministic, which is usually expected from computer systems.
- **Mutual Exclusion**: To avoid race conditions, threads should use some kind of mutual exclusion primitives. Mutual exclusion guarantees that only a single thread ever enters a critical section at a time, thus avoiding races and resulting in deterministic program outputs.

## Questions & Answers

1. Let’s examine a simple program, “loop.s”. First, just read and understand it. Then, run it with these arguments `./x86.py -t 1-p loop.s -i 100 -R dx` This specifies a single thread, an interrupt every 100 instructions, and tracing of register `%dx`. What will `%dx` be during the run? Use the `-c` flag to check your answers; the answers, on the left, show the value of the register (or memory value) after the instruction on the right has run.
    
    The `%dx` value will be -1 as soon as it starts running
    
    ```bash
       dx          Thread 0         
        0   
       -1   1000 sub  $1,%dx
       -1   1001 test $0,%dx
       -1   1002 jgte .top
       -1   1003 halt
    ```
    

1. Same code, different flags: `./x86.py -p loop.s -t 2 -i 100 -a dx=3,dx=3 -R dx` This specifies two threads, and initializes each `%dx` to 3. What values will `%dx` see? Run with `-c` to check. Does the presence of multiple threads affect your calculations? Is there a race in this code?
    
    Both the threads will reduce the “dx” value 3 until it is less than 0. The presence of multiple threads do not affect the calculations. There is no race in this code.
    
    ```bash
       dx          Thread 0                Thread 1         
        3   
        2   1000 sub  $1,%dx
        2   1001 test $0,%dx
        2   1002 jgte .top
        1   1000 sub  $1,%dx
        1   1001 test $0,%dx
        1   1002 jgte .top
        0   1000 sub  $1,%dx
        0   1001 test $0,%dx
        0   1002 jgte .top
       -1   1000 sub  $1,%dx
       -1   1001 test $0,%dx
       -1   1002 jgte .top
       -1   1003 halt
        3   ----- Halt;Switch -----  ----- Halt;Switch -----  
        2                            1000 sub  $1,%dx
        2                            1001 test $0,%dx
        2                            1002 jgte .top
        1                            1000 sub  $1,%dx
        1                            1001 test $0,%dx
        1                            1002 jgte .top
        0                            1000 sub  $1,%dx
        0                            1001 test $0,%dx
        0                            1002 jgte .top
       -1                            1000 sub  $1,%dx
       -1                            1001 test $0,%dx
       -1                            1002 jgte .top
       -1                            1003 halt
    
    ```
    

1. Run this: `./x86.py -p loop.s -t 2 -i 3 -r -R dx -a dx=3,dx=3` This makes the interrupt interval small/random; use different seeds (`-s`) to see different inter leavings. Does the interrupt frequency change anything?
    
    Yes, the small/random interrupt can and will cause race conditions most of the times.
    
2. **Now, a different program, looping-race-nolock.s, which accesses a shared variable located at address 2000; we’ll call this variable value. Run it with a single thread to confirm your understanding: `./x86.py -p looping-race-nolock.s -t 1 -M 2000` What is value (i.e., at memory address 2000) throughout the run? Use `-c` to check.**
    
    The value will be 1 throughout the run, as there is no race condition and number of threads is one.
    
3. **Run with multiple iterations/threads: `./x86.py -p looping-race-nolock.s -t 2 -a bx=3 -M 2000` Why does each thread loop three times? What is final value of value?**
    
    The each thread loops for three times because after each loop the program will decrease the value of bx by 1 and exits when the value of bx reaches 0. bx starts from 3.
    
4. **Run with random interrupt intervals: `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 0` with different seeds (-s 1, -s 2, etc.) Can you tell by looking at the thread interleaving what the final value of value will be? Does the timing of the interrupt matter? Where can it safely occur? Where not? In other words, where is the critical section exactly?**
    
    Yes the timing of the interrupt matter. The critical region is this piece of code:
    
    ```bash
    1004 test $0, %bx
    1005 jgt .top
    ```
    
5. **Now examine fixed interrupt intervals: `./x86.py -p looping-race-nolock.s -a bx=1 -t 2 -M 2000 -i 1` What will the final value of the shared variable value be? What about when you change `-i 2`, `-i 3`, etc.? For which interrupt intervals does the program give the “correct” answer?**
    
    The shared value will be 1. for -i 3 the program gives the “correct answer because it will interrupt after pushing the value back to memory, hence the other thread will get the updated value.
    
6. **Run the same for more loops (e.g., set -a bx=100). What interrupt intervals (-i) lead to a correct outcome? Which intervals are surprising?**
    
    -i 3 interrupt intervals will lead to a correct outcome. All the intervals after 7 are surprising as the value is  less than 200 but cannot be predicted.
    
7. **One last program: wait-for-me.s. Run: `./x86.py -p wait-for-me.s -a ax=1,ax=0 -R ax -M 2000` This sets the `%ax` register to 1 for thread 0, and 0 for thread 1, and watches `%ax` and memory location 2000. How should the code behave? How is the value at location 2000 being used by the threads? What will its final value be?**
    
    This code allows one thread to signal the other that some condition is met by writing to the shared 2000 location.So thread 0 will store 1 to 2000, allowing thread 1 to exit the loop and halt. The final value at 2000 will be 1.
    
8. **Now switch the inputs: ./x86.py -p wait-for-me.s -a ax=0,ax=1 -R ax -M 2000 How do the threads behave? What is thread 0 doing? How would changing the interrupt interval (e.g.,
-i 1000, or perhaps to use random intervals) change the trace outcome? Is the program efficiently using the CPU?**
    
    Now thread 0 is the waiter and thread 1 is the signaller.
    Thread 0 (waiter) will loop checking 2000 until thread 1 stores a 1 there.
    Thread 1 (signaller) will store 1 to 2000 and then halt.
    However, since thread 1 runs first and stores 1 to 2000 immediately, thread 0 may loop for a very long time unnecessarily burning CPU cycles before getting to run and seeing the 1 value.
    Changing the interrupt interval with -i 1000 or using random intervals -i 4 -r would make thread 0 get a chance to run sooner after thread 1 signals. But it's still an inefficient way to synchronize as the waiter burns CPU looping.
