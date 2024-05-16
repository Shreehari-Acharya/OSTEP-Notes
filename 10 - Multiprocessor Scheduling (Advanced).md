## Introduction to Multiprocessor Scheduling

- Multiprocessor systems have become common with rise of multi-core CPUs
- Need to rewrite single-threaded applications to run in parallel to take advantage of multiple CPUs
- Operating systems face new problem of how to schedule jobs/threads on multiple CPUs

## Background: Multiprocessor Architecture

- Systems with single CPU use caches to exploit temporal/spatial locality
- With multiple CPUs sharing main memory, caching is more complex due to cache coherence issue
- Hardware solutions like bus snooping ensure coherent view of shared memory

## Don't Forget Synchronization

- When accessing shared data structures across CPUs, need to use mutual exclusion (locks)
- Failure to do so can lead to race conditions/incorrect results
- But locks can become a bottleneck/scalability issue with many CPUs contending

## Cache Affinity

- Advantage to executing a process on same CPU if its working set is cached there
- Scheduler should try to preserve cache affinity when possible

## Single Queue Multiprocessor Scheduling (SQMS)

- Simple to implement by reusing uniprocessor scheduling algorithms
- But has scalability issues due to locking on shared queue
- And does not inherently preserve cache affinity

## Multi-Queue Multiprocessor Scheduling (MQMS)

- Has a separate queue per CPU to avoid contention
- Naturally provides cache affinity by not migrating processes
- But suffers from potential load imbalance across queues
- Work stealing can redistribute load but has overhead

## Linux Implementations

- O(1) scheduler - multiple queues, priority-based
- Completely Fair Scheduler (CFS) - multiple queues, proportional-share
- BFS - single queue, Earliest Eligible Virtual Deadline First algorithm

## Tradeoffs

- SQMS is simple but doesn't scale well and lacks affinity
- MQMS scales better and has affinity but load balancing is an issue
- No universally optimal solution, depends on workload characteristics

## Questions & Answers

**1. To start things off, let’s learn how to use the simulator to study how to build an effective multi-processor scheduler. The first simulation will run just one job, which has a run-time of 30, and a working-set size of 200. Run this job (called job ’a’ here) on one simulated CPU as follows: `./multi.py -n 1 -L a:30:200`. How long will it take to complete? Turn on the -c flag to see a final answer, and the -t flag to see a tick-by-tick trace of the job and how it is scheduled.**

**Ans:** Since there is only one job and the default policy is centralized scheduling queue, It will take 30 Units of time to complete.

**2. Now increase the cache size so as to make the job’s working set (size=200) fit into the cache (which, by default, is size=100); for example, run `./multi.py -n 1 -L a:30:200 -M 300`. Can you predict how fast the job will run once it fits in cache? (hint: remember the key parameter of the warm rate, which is set by the -r flag) Check your answer by running with the solve flag (-c) enabled.**

**Ans:** Working set = 200 ; cache size = 300(increased with -M flag) warmup_time = 10 ; warm rate = 2x (default) 

the first 10units are needed for warmup and then instead of taking 20units, due to warm rate it will only take 20/2 = 10. Therefore total time = 10 + 10 = 20

**3. One cool thing about multi.py is that you can see more detail about what is going on with different tracing flags. Run the same simulation as above, but this time with time left tracing enabled (-T). This flag shows both the job that was scheduled on a CPU at each time step, as well as how much run-time that job has left after each tick has run. What do you notice about how that second column decreases?**

```bash
User@Linux:~/ostep-homework/cpu-sched-multi$ python3 ./multi.py -n 1 -L a:30:200 -M 300 -T
ARG seed 0
ARG job_num 3
ARG max_run 100
ARG max_wset 200
ARG job_list a:30:200
ARG affinity 
ARG per_cpu_queues False
ARG num_cpus 1
ARG quantum 10
ARG peek_interval 30
ARG warmup_time 10
ARG cache_size 300
ARG random_order False
ARG trace False
ARG trace_time True
ARG trace_cache False
ARG trace_sched False
ARG compute False

Job name:a run_time:30 working_set_size:200

Scheduler central queue: ['a']

   0   a [ 29]      
   1   a [ 28]      
   2   a [ 27]      
   3   a [ 26]      
   4   a [ 25]      
   5   a [ 24]      
   6   a [ 23]      
   7   a [ 22]      
   8   a [ 21]      
   9   a [ 20]      
----------------
  10   a [ 18]      
  11   a [ 16]      
  12   a [ 14]      
  13   a [ 12]      
  14   a [ 10]      
  15   a [  8]      
  16   a [  6]      
  17   a [  4]      
  18   a [  2]      
  19   a [  0]      

```

**Ans:** We can clearly see that after 10units the time starts to decrease by 2 units at a time.

**4.Now add one more bit of tracing, to show the status of each CPU cache for each job, with the -C flag. For each job, each cache will either show a blank space (if the cache is cold for that job) or a ’w’ (if the cache is warm for that job). At what point does the cache become warm for job ’a’ in this simple example? What happens as you change the warmup time parameter (-w) to lower or higher values than the default?**

```bash
User@Linux:~/ostep-homework/cpu-sched-multi$ python3 ./multi.py -n 1 -L a:30:200 -M 300 -C
ARG seed 0
ARG job_num 3
ARG max_run 100
ARG max_wset 200
ARG job_list a:30:200
ARG affinity 
ARG per_cpu_queues False
ARG num_cpus 1
ARG quantum 10
ARG peek_interval 30
ARG warmup_time 10
ARG cache_size 300
ARG random_order False
ARG trace False
ARG trace_time False
ARG trace_cache True
ARG trace_sched False
ARG compute False

Job name:a run_time:30 working_set_size:200

Scheduler central queue: ['a']

   0   a cache[ ]     
   1   a cache[ ]     
   2   a cache[ ]     
   3   a cache[ ]     
   4   a cache[ ]     
   5   a cache[ ]     
   6   a cache[ ]     
   7   a cache[ ]     
   8   a cache[ ]     
   9   a cache[w]     
-------------------
  10   a cache[w]     
  11   a cache[w]     
  12   a cache[w]     
  13   a cache[w]     
  14   a cache[w]     
  15   a cache[w]     
  16   a cache[w]     
  17   a cache[w]     
  18   a cache[w]     
  19   a cache[w]     
```

**Ans:** We can see that warmup starts at the 10th unit of times. as it is the default value. If we decrease the warmup time; the job will finish faster, and if increased; it will take longer to finish the job

**5.At this point, you should have a good idea of how the simulator works for a single job running on a single CPU. But hey, isn’t this a multi-processor CPU scheduling chapter? Oh yeah! So let’s start working with multiple jobs. Specifically, let’s run the following three jobs on a two-CPU system (i.e., type `./multi.py -n 2 -L a:100:100,b:100:50,c:100:50`) Can you predict how long this will take, given a round-robin centralized scheduler? Use -c to see if you were right, and then dive down into details with -t to see a step-by-step and then -C to see whether caches got warmed effectively for these jobs. What do you notice?**

**Ans:** Given a round robin centralized scheduler and 2 CPU it  will take total runtime / Number of CPU. Therefore 300/2 = 150

Although cache was warmed, it couldn't synchronize the cache with other CPU, hence it did not decrease the run time.

**6. Now we’ll apply some explicit controls to study cache affinity, as described in the chapter. To do this, you’ll need the -A flag. This flag can be used to limit which CPU's the scheduler can place a particular job upon. In this case, let’s use it to place jobs ’b’ and ’c’ on CPU 1, while restricting ’a’ to CPU 0. This magic is accomplished by typing this `./multi.py -n 2 -L a:100:100,b:100:50,c:100:50 -A a:0,b:1,c:1` ; don’t forget to turn on various tracing options to see what is really happening! Can you predict how fast this version will run? Why does it do better? Will other combinations of ’a’, ’b’, and ’c’ onto the two processors run faster or slower?**

**Ans:** CPU 0 has Job a with runtime of 100: first 10 units will take for the warmup of cache and the rest 90 units will be done in 45units as the warm rate is 2x. Total of 55 units.

CPU 1 has Job ‘b’ and ‘c’ with runtime of 100 each: 10 + 10 units will take for the warmup of cache and the rest 90 + 90 will be done is 45+45 units of time. total of 110 units

As there are 2 CPU, the CPU taking the highest time will be the total run time. i.e 110 Units

The other combinations of ‘a’ ‘b’ and ‘c’ onto two processors run slower, because ‘b’ and ‘c’ has a cache size of 50 each. Hence its efficient to run both of them in a single CPU which has a cache size of 100.

**7. One interesting aspect of caching multiprocessors is the opportunity for better-than-expected speed up of jobs when using multiple CPU's (and their caches) as compared to running jobs on a single processor. Specifically, when you run on N CPU's, sometimes you can speed up by more than a factor of N , a situation entitled super-linear speedup. To experiment with this, use the job description here (`-L a:100:100,b:100:100,c:100:100`) with a small cache (-M 50) to create three jobs. Run this on systems with 1, 2, and 3 CPU's (-n 1, -n 2, -n 3). Now, do the same, but with a larger per-CPU cache of size 100. What do you notice about performance as the number of CPU's scales? Use -c to confirm your guesses, and other tracing flags to dive even deeper**

**Ans:**

`User@Linux:~/ostep-homework/cpu-sched-multi$ python3 ./multi.py -L a:100:100,b:100:100,c:100:100 -n 1 -M 50`

```bash
Finished time 300
Per-CPU stats
  CPU 0  utilization 100.00 [ warm 0.00 ]
```

`User@Linux:~/ostep-homework/cpu-sched-multi$ python3 ./multi.py -L a:100:100,b:100:100,c:100:100 -n 2 -M 50`

```bash
Finished time 150
Per-CPU stats
  CPU 0  utilization 100.00 [ warm 0.00 ]
  CPU 1  utilization 100.00 [ warm 0.00 ]
```

`User@Linux:~/ostep-homework/cpu-sched-multi$ python3 ./multi.py -L a:100:100,b:100:100,c:100:100 -n 3 -M 50`

```bash
Finished time 100
Per-CPU stats
  CPU 0  utilization 100.00 [ warm 0.00 ]
  CPU 1  utilization 100.00 [ warm 0.00 ]
  CPU 2  utilization 100.00 [ warm 0.00 ]
```

`User@Linux:~/ostep-homework/cpu-sched-multi$ python3 ./multi.py -L a:100:100,b:100:100,c:100:100 -n 1 -M 100`

```bash
Finished time 300
Per-CPU stats
  CPU 0  utilization 100.00 [ warm 0.00 ]
```

`User@Linux:~/ostep-homework/cpu-sched-multi$ python3 ./multi.py -L a:100:100,b:100:100,c:100:100 -n 2 -M 100`

```bash
Finished time 150
Per-CPU stats
  CPU 0  utilization 100.00 [ warm 0.00 ]
  CPU 1  utilization 100.00 [ warm 0.00 ]
```

`User@Linux:~/ostep-homework/cpu-sched-multi$ python3 ./multi.py -L a:100:100,b:100:100,c:100:100 -n 3 -M 100`

```bash
Finished time 55
Per-CPU stats
  CPU 0  utilization 100.00 [ warm 81.82 ]
  CPU 1  utilization 100.00 [ warm 81.82 ]
  CPU 2  utilization 100.00 [ warm 81.82 ]
```

**8. One other aspect of the simulator worth studying is the per-CPU scheduling option, the -p flag. Run with two CPU's again, and this three job configuration (`-L a:100:100,b:100:50,c:100:50`). How does this option do, as opposed to the hand-controlled affinity limits you put in place above? How does performance change as you alter the ’peek interval’ (-P) to lower or higher values? How does this per-CPU approach work as the number of CPU's scales?**

**Ans:** This options performs better than the hand-controlled affinity limits.

The performance increases no matter we increase or decrease the values.

It will distribute the Jobs as the number of CPU scales.
