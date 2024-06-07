# Semaphores

- Introduced by Edsger Dijkstra as a synchronization primitive for concurrent programming
- Can be used as both locks and condition variables

## How to Use Semaphores

- Semaphore is an object with an integer value and two routines: `sem_wait()` and `sem_post()`
- `sem_wait()` decrements the value and blocks if value is negative
- `sem_post()` increments the value and wakes a waiting thread if any

## Binary Semaphores (Locks)

- Initialize semaphore to 1 to use as a lock
- `sem_wait()` acquires the lock, `sem_post()` releases it

## Semaphores for Ordering

- Initialize semaphore to 0
- One thread does `sem_wait()` to block until another calls `sem_post()`
- Used to wait for an event/condition to become true

## Producer/Consumer (Bounded Buffer) Problem

### First Attempt (Incorrect)

- Use two semaphores: `empty` and `full`
- Producer waits on `empty`, then puts data and posts `full`
- Consumer waits on `full`, then gets data and posts `empty`
- **Race condition** when multiple producers write to the same buffer index

### Solution: Adding Mutual Exclusion (Correctly)

- Add a mutex lock around the critical section of `put()` and `get()`
- Solves the race condition

## Reader-Writer Locks

- Allow multiple readers or a single writer
- Uses three semaphores: `lock`, `writelock`, and a counter `readers`
- Readers acquire `lock`, increment `readers`, acquire `writelock` if first reader
- Writers acquire `writelock`
- **Starvation** possible if many readers keep entering

## Dining Philosophers

- Classic concurrency problem involving 5 philosophers sharing 5 forks
- Each philosopher needs 2 forks to eat
- Broken solution: Deadlock if all grab left fork first
- Solution: Philosopher 4 grabs right fork first to break the cycle

## Thread Throttling

- Use a semaphore to limit the number of threads executing a part of code
- Prevents overloading system resources like memory

## Summary

- Semaphores are powerful and flexible for concurrent programming
- Many classic problems can be solved using semaphores
- Be careful when generalizing semaphores from locks and condition variables

## Questions & Answers:

1. The first problem is just to implement and test a solution to the fork/join problem, as described in the text. Even though this solution is described in the text, the act of typing it in on your own is worthwhile; even Bach would rewrite Vivaldi, allowing one soon-to-be master to learn from an existing one. See fork-join.c for details. Add the call sleep(1) to the child to ensure it is working.
    
    ```c
    #include <stdio.h>
    #include <unistd.h>
    #include <pthread.h>
    #include "common_threads.h"
    
    sem_t s; 
    
    void *child(void *arg) {
        sleep(1);
        printf("child\n");
        sem_post(&s); // use semaphore here
        return NULL;
    }
    
    int main(int argc, char *argv[]) {
        pthread_t p;
        printf("parent: begin\n");
        sem_init(&s, 0, 0); // init semaphore here
        Pthread_create(&p, NULL, child, NULL);
        sem_wait(&s);// use semaphore here
        printf("parent: end\n");
        return 0;
    }
    
    ```
    
2. Let’s now generalize this a bit by investigating the rendezvous problem. The problem is as follows: you have two threads, each of which are about to enter the rendezvous point in the code. Neither should exit this part of the code before the other enters it. Consider using two semaphores for this
task, and see rendezvous.c for details.
    
    ```c
    #include <stdio.h>
    #include <unistd.h>
    #include "common_threads.h"
    
    // If done correctly, each child should print their "before" message
    // before either prints their "after" message. Test by adding sleep(1)
    // calls in various locations.
    
    sem_t s1, s2;
    
    void *child_1(void *arg) {
        sleep(1);
        printf("child 1: before\n");
        sem_post(&s1);
        sem_wait(&s2);// what goes here?
        sleep(1);
        printf("child 1: after\n");
        return NULL;
    }
    
    void *child_2(void *arg) {
        sleep(1);
        printf("child 2: before\n");
        sem_post(&s2);
        sem_wait(&s1);// what goes here?
        sleep(1);
        printf("child 2: after\n");
        return NULL;
    }
    
    int main(int argc, char *argv[]) {
        pthread_t p1, p2;
        printf("parent: begin\n");
        sem_init(&s1,0,0);
        sem_init(&s2,0,0); // init semaphores here
        Pthread_create(&p1, NULL, child_1, NULL);
        Pthread_create(&p2, NULL, child_2, NULL);
        Pthread_join(p1, NULL);
        Pthread_join(p2, NULL);
        printf("parent: end\n");
        return 0;
    }
    
    ```
    
3. Now go one step further by implementing a general solution to barrier synchronization. Assume there are two points in a sequential piece of code, called P1 and P2. Putting a barrier between P1 and P2 guarantees that all threads will execute P1 before any one thread executes P2. Your task: write the code to implement a barrier() function that can be used in this manner. It is safe to assume you know N (the total number of threads in the running program) and that all N threads will try to enter the barrier. Again, you should likely use two semaphores to achieve the solution, and some other integers to count things. See barrier.c for details.
4. Now let’s solve the reader-writer problem, also as described in the text. In this first take, don’t worry about starvation. See the code in reader-writer.c for details. Add sleep() calls to your code to demonstrate it works as you expect. Can you show the existence of the starvation problem?
5. Let’s look at the reader-writer problem again, but this time, worry about starvation. How can you ensure that all readers and writers eventually make progress? See reader-writer-nostarve.c for details.
6. Use semaphores to build a no-starve mutex, in which any thread that tries to acquire the mutex will eventually obtain it. See the code in mutex-nostarve.c for more information.
