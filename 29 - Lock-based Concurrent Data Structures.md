## The Challenge

- When given a data structure, how do we add locks to make it thread-safe and achieve high performance with many concurrent accesses?

## Concurrent Counters

- **Simple But Not Scalable**
    - Adding a single lock around increment/decrement operations (Figure 29.2)
    - Poor performance scaling as number of threads increases (Figure 29.5 - 'Precise')
- **Scalable Counting**
    - Approximate Counter
        - Represents a logical counter using per-CPU local counters and a global counter
        - Threads update local counters, periodically transfer to global counter
        - Allows scalability by reducing contention on a single lock
        - Tunable accuracy/performance tradeoff via threshold S (Figures 29.3, 29.4, 29.6)
    - Crucial for some workloads on Linux to scale well on multicores

## Concurrent Linked Lists

- Simple approach: Single lock around list operations (Figure 29.7)
- Can be rewritten to reduce lock acquires/releases on rare failure paths (Figure 29.8)
- Hand-over-hand locking: One lock per node, acquire next node's lock before releasing current
    - Enables more concurrency, but high overhead negates benefits in practice

## Concurrent Queues

- Michael & Scott Queue (Figure 29.9)
    - Two locks: one for head, one for tail
    - Enqueue acquires tail lock, dequeue acquires head lock
    - Enables concurrency between enqueue and dequeue operations

## Concurrent Hash Tables

- Hash table with one lock per bucket (Figure 29.10)
- Scales very well compared to a single lock (Figure 29.11)

## Tips and General Advice

- Start with a simple approach (single big lock), optimize only if needed
- Be wary of complex control flows that require lock acquires/releases
- Enabling more concurrency doesn't necessarily improve performance
- Knuth's law: "Premature optimization is the root of all evil"
- Measure alternatives before claiming one is faster

The key takeaways are to start simple, measure performance, and optimize only if needed, while being careful about complex concurrency control that can introduce bugs. 

## Questions & Answers:

1. We’ll start by redoing the measurements within this chapter. Use the call `gettimeofday()` to measure time within your program. How accurate is this timer? What is the smallest interval it can measure? Gain confidence in its workings, as we will need it in all subsequent questions. You can also look into other timers, such as the cycle counter available on x86 via the rdtsc instruction.
    - `gettimeofday()` is reasonably accurate, providing time in microseconds (millionths of a second) on most systems.
    - The smallest interval it can measure is typically in the nanosecond range for modern systems
2. Now, build a simple concurrent counter and measure how long it takes to increment the counter many times as the number of threads increases. How many CPUs are available on the system you are using? Does this number impact your measurements at all?
    
    C code of a simple concurrent counter:
    
    ```c
    #include <sys/time.h>
    #include <pthread.h>
    #include <stdio.h>
    #include <stdlib.h>
    
    typedef struct __counter_t {
        int value;
        pthread_mutex_t lock;
    } counter_t;
    
    void init(counter_t *c) {
        c->value = 0;
        pthread_mutex_init(&c->lock, NULL);
    }
    
    void increment(counter_t *c) {
        pthread_mutex_lock(&c->lock);
        c->value++;
        pthread_mutex_unlock(&c->lock);
    }
    
    // A function to increment the counter for large times.
    void *thread_func(void *arg) {
        counter_t *c = (counter_t *)arg;
        for (int i = 0; i < 1000000; i++) { // looping for 10Lakhs/1 Million times
            increment(c);
        }
        return NULL;
    }
    
    int main(int argc, char* argv[]) {
        if(argc < 2){
    	 printf("Usage: ./counter number_of_threads\n");
    	 exit(0);
        }
        counter_t c;
        init(&c);
        int threads_cnt = atoi(argv[1]);
        struct timeval start, end;
    
        // create number of threads specified by argv and time their execution
        pthread_t threads[threads_cnt];
        gettimeofday(&start, NULL);
        for (int i = 0; i < threads_cnt; i++) {
            pthread_create(&threads[i], NULL, thread_func, (void *)&c);
        }
        for (int i = 0; i < threads_cnt; i++) {
            pthread_join(threads[i], NULL);
        }
        gettimeofday(&end, NULL);
    
        // calculate and print time taken
        long long microseconds = (end.tv_sec - start.tv_sec) * 1000000LL + (end.tv_usec - start.tv_usec);
        double seconds = (double)microseconds / 1000000.0;
    
        printf("Time taken: %lld microseconds (%.6f seconds)\n", microseconds, seconds);
    
        return 0;
    };
    
    ```
    
    Compile the program using `gcc counter.c -pthread counter` 
    
    To find out the number of CPU’s in your  system, you can use the following command
    
    `User@Linux: lscpu` 
    
    Yes it impacts negetively, the more threads we use, the longer it takes to finish the task.
    
    Some execution examples:
    
    ```bash
    User@Linux:~/programs$ ./counter 4
    Time taken: 255575 microseconds (0.255575 seconds)
    User@Linux:~/programs$ ./counter 8
    Time taken: 482870 microseconds (0.482870 seconds)
    User@Linux:~/programs$ ./counter 12
    Time taken: 787183 microseconds (0.787183 seconds)
    User@Linux:~/programs$ ./counter 16
    Time taken: 1046866 microseconds (1.046866 seconds)
    ```
    
3. Next, build a version of the approximate counter. Once again, measure its performance as the number of threads varies, as well as the threshold. Do the numbers match what you see in the chapter?
    
    C code for approximate counter:
    
    ```c
    #include <pthread.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <sys/time.h>
    
    #define NUMCPUS 8 // Change this based on your system
    int NUM_THREADS; // A global variable to store the number of threads from input
    
    typedef struct __counter_t {
        int global;
        pthread_mutex_t glock;
        int local[NUMCPUS];
        pthread_mutex_t llock[NUMCPUS];
        int threshold;
    } counter_t;
    
    void init(counter_t *c, int threshold) {
        c->threshold = threshold;
        c->global = 0;
        pthread_mutex_init(&c->glock, NULL);
        for (int i = 0; i < NUMCPUS; i++) {
            c->local[i] = 0;
            pthread_mutex_init(&c->llock[i], NULL);
        }
    }
    
    void update(counter_t *c, int threadID, int amt) {
        int cpu = threadID % NUMCPUS;
        pthread_mutex_lock(&c->llock[cpu]);
        c->local[cpu] += amt;
        if (c->local[cpu] >= c->threshold) {
            pthread_mutex_lock(&c->glock);
            c->global += c->local[cpu];
            pthread_mutex_unlock(&c->glock);
            c->local[cpu] = 0;
        }
        pthread_mutex_unlock(&c->llock[cpu]);
    }
    
    int get(counter_t *c) {
        pthread_mutex_lock(&c->glock);
        int val = c->global;
        for (int i = 0; i < NUMCPUS; i++) {
            val += c->local[i];
        }
        pthread_mutex_unlock(&c->glock);
        return val;
    }
    #define ITERS_PER_THREAD 100000000
    
    void *thread_func(void *arg) {
        counter_t *c = (counter_t *)arg;
        int threadID = pthread_self() % NUM_THREADS;
        for (int i = 0; i < ITERS_PER_THREAD; i++) {
            update(c, threadID, 1);
        }
        return NULL;
    }
    
    int main(int argc, char *argv[]) {
        if (argc != 3) {
            printf("Usage: %s <threshold> <number_of_threads>\n", argv[0]);
            return 1;
        }
    
        int threshold = atoi(argv[1]);
        NUM_THREADS = atoi(argv[2]);
        counter_t c;
        init(&c, threshold);
    
        pthread_t threads[NUM_THREADS];
    
        struct timeval start_time, end_time;
        gettimeofday(&start_time, NULL); // Start time
    
        for (int i = 0; i < NUM_THREADS; i++) {
            pthread_create(&threads[i], NULL, thread_func, (void *)&c);
        }
    
        for (int i = 0; i < NUM_THREADS; i++) {
            pthread_join(threads[i], NULL);
        }
    
        gettimeofday(&end_time, NULL); // End time
    
        int final_count = get(&c);
        printf("Final count: %d\n", final_count);
    
        long long microseconds = (end_time.tv_sec - start_time.tv_sec) * 1000000LL +
                                 (end_time.tv_usec - start_time.tv_usec);
        double seconds = (double)microseconds / 1000000.0;
    
        printf("Time taken: %lld microseconds (%.6f seconds)\n", microseconds, seconds);
    
        return 0;
    }
    
    ```
I tried to do the other questions, but its too tuff... :( <br> 

4. Build a version of a linked list that uses hand-over-hand locking [MS04], as cited in the chapter. You should read the paper first to understand how it works, and then implement it. Measure its
performance. When does a hand-over-hand list work better than a standard list as shown in the chapter? <br>

5. Pick your favorite data structure, such as a B-tree or other slightly more interesting structure. Implement it, and start with a simple locking strategy such as a single lock. Measure its performance as the number of concurrent threads increases.<br>

6. Finally, think of a more interesting locking strategy for this favorite data structure of yours. Implement it, and measure its performance. How does it compare to the straightforward locking approach?
