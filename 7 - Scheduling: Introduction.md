# Scheduling: Introduction
### Workload Assumptions:

Initially, simplifying assumptions are made: all jobs run for the same time, arrive simultaneously, run to completion without I/O, and their run-times are known. These unrealistic assumptions are gradually relaxed later.

### Scheduling Metrics:

The primary metric used is turnaround time (job completion time - arrival time). Other metrics like fairness (Jain's Fairness Index) are also relevant but turnaround time is the focus.

### Summary of FIFO, SJF, and STCF:

- FIFO (First In First Out) is simple but suffers from the convoy effect with varying job lengths.
- SJF (Shortest Job First) is optimal for minimizing average turnaround time when jobs arrive together.
- STCF (Shortest Time-to-Completion First) adds preemption to SJF and is optimal when jobs can arrive at any time.

### Response Time and Round Robin:

Response time (time to first scheduling) is important for interactive systems. RR (Round Robin) optimizer for response time by running jobs in small time slices but is poor for turnaround time. There is a trade-off between optimizing turnaround time vs response time.

### Basic Formula’s:

**T***turnaround* = **T***completion* − **T***arrival* 

**T***response* = **T***firstrun* − **T***arrival* 

The scheduler treats CPU bursts between I/Os as jobs to overlap CPU and I/O operations. Finally, the assumption of knowing job run-times is dropped, motivating techniques like **multi-level feedback queues covered next.**


## Questions:
### 1. Compute the response time and turnaround time when running three jobs of length 200 with the SJF and FIFO schedulers.

  We will use the formula **T***turnaround* = **T***completion* − **T***arrival*  & **T***response* = **T***firstrun* − **T***arrival* 

  **Scheduler - SJF**

  ```bash
User@Linux:~/ostep-homework/cpu-sched$ ./scheduler.py -j 3 -l 200,200,200 -p SJF
ARG policy SJF
ARG jlist 200,200,200

Here is the job list, with the run time of each job: 
  Job 0 ( length = 200.0 )
  Job 1 ( length = 200.0 )
  Job 2 ( length = 200.0 )

Compute the turnaround time, response time, and wait time for each job.
When you are done, run this program again, with the same arguments,
but with -c, which will thus provide you with the answers. You can use
-s <somenumber> or your own job list (-l 10,15,20 for example)
to generate different problems for yourself.
  ```

| Jobs | Tcompletion - Tarrival = Tturnaround | Tfirstrun - Tarrival = Tresponse | Wait time |
| --- | --- | --- | --- |
| Job 0 | 200.0 - 0 = 200.0 | 0 - 0 = 0.0 | 0 |
| Job 1 | 400.0 - 0 = 400.0  | 200.0 - 0 = 200.0 | 200.0 |
| Job 2 | 600.0 - 0 = 600.0 | 400.0 - 0 = 400.0 | 400.0 |
| Average | 1200.0 / 3.0 = 400.0 | 600.0 / 3.0 = 200.0 | 600.0 / 3.0 = 200.0 |

  **Scheduler - FIFO**

```bash
User@Linux:~/ostep-homework/cpu-sched$ ./scheduler.py -j 3 -l 200,200,200 -p FIFO
ARG policy FIFO
ARG jlist 200,200,200

Here is the job list, with the run time of each job: 
  Job 0 ( length = 200.0 )
  Job 1 ( length = 200.0 )
  Job 2 ( length = 200.0 )

Compute the turnaround time, response time, and wait time for each job.
When you are done, run this program again, with the same arguments,
but with -c, which will thus provide you with the answers. You can use
-s <somenumber> or your own job list (-l 10,15,20 for example)
to generate different problems for yourself.
```

| Jobs | Tcompletion - Tarrival = Tturnaround | Tfirstrun - Tarrival = Tresponse | Wait time |
| --- | --- | --- | --- |
| Job 0 | 200.0 - 0 = 200.0 | 0 - 0 = 0.0 | 0 |
| Job 1 | 400.0 - 0 = 400.0  | 200.0 - 0 = 200.0 | 200.0 |
| Job 2 | 600.0 - 0 = 600.0 | 400.0 - 0 = 400.0 | 400.0 |
| Average | 1200.0 / 3.0 = 400.0 | 600.0 / 3.0 = 200.0 | 600.0 / 3.0 = 200.0 |

### 2. Now do the same but with jobs of different lengths: 100, 200, and 300

**Scheduler - SJF**

```bash
User@Linux:~/ostep-homework/cpu-sched$ ./scheduler.py -j 3 -l 100,200,300 -p SJC
ARG policy SJC
ARG jlist 100,200,300

Here is the job list, with the run time of each job: 
  Job 0 ( length = 100.0 )
  Job 1 ( length = 200.0 )
  Job 2 ( length = 300.0 )

Compute the turnaround time, response time, and wait time for each job.
When you are done, run this program again, with the same arguments,
but with -c, which will thus provide you with the answers. You can use
-s <somenumber> or your own job list (-l 10,15,20 for example)
to generate different problems for yourself.
```

| Jobs | Tcompletion - Tarrival = Tturnaround | Tfirstrun - Tarrival = Tresponse | Wait time |
| --- | --- | --- | --- |
| Job 0 | 100.0 - 0 = 100.0 | 0 - 0 = 0.0 | 0 |
| Job 1 | 300.0 - 0 = 300.0  | 100.0 - 0 = 100.0 | 100.0 |
| Job 2 | 600.0 - 0 = 600.0 | 300.0 - 0 = 300.0 | 300.0 |
| Average | 1000.0 / 3.0 = 333.33 | 400.0 / 3.0 = 133.33 | 400.0 / 3.0 = 133.33 |

**Scheduler FIFO**

```bash
user@Linux:~/ostep-homework/cpu-sched$ ./scheduler.py -j 3 -l 100,200,300 -p FIFO 
ARG policy FIFO
ARG jlist 100,200,300

Here is the job list, with the run time of each job: 
  Job 0 ( length = 100.0 )
  Job 1 ( length = 200.0 )
  Job 2 ( length = 300.0 )

Compute the turnaround time, response time, and wait time for each job.
When you are done, run this program again, with the same arguments,
but with -c, which will thus provide you with the answers. You can use
-s <somenumber> or your own job list (-l 10,15,20 for example)
to generate different problems for yourself.
```

| Jobs | Tcompletion - Tarrival = Tturnaround | Tfirstrun - Tarrival = Tresponse | Wait time |
| --- | --- | --- | --- |
| Job 0 | 100.0 - 0 = 100.0 | 0 - 0 = 0.0 | 0 |
| Job 1 | 300.0 - 0 = 300.0  | 100.0 - 0 = 100.0 | 100.0 |
| Job 2 | 600.0 - 0 = 600.0 | 300.0 - 0 = 300.0 | 300.0 |
| Average | 1000.0 / 3.0 = 333.33 | 400.0 / 3.0 = 133.33 | 400.0 / 3.0 = 133.33 |

### 3. Now do the same, but also with the RR scheduler and a time-slice of 1.

```bash
User@Linux:~/ostep-homework/cpu-sched$ ./scheduler.py -j 3 -l 100,200,300 -p RR -q 1
ARG policy RR
ARG jlist 100,200,300 

Here is the job list, with the run time of each job: 
  Job 0 ( length = 100.0 )
  Job 1 ( length = 200.0 )
  Job 2 ( length = 300.0 )

Compute the turnaround time, response time, and wait time for each job.
When you are done, run this program again, with the same arguments,
but with -c, which will thus provide you with the answers. You can use
-s <somenumber> or your own job list (-l 10,15,20 for example)
to generate different problems for yourself.
```

| Jobs | Tcompletion - Tarrival = Tturnaround | Tfirstrun - Tarrival = Tresponse | Wait time |
| --- | --- | --- | --- |
| Job 0 | 298.0 - 0 = 298.0 | 0 - 0 = 0.0 | 0 |
| Job 1 | 499.0 - 0 = 499.0  | 1.0 - 0 = 1.0 | 100.0 |
| Job 2 | 600.0 - 0 = 600.0 | 2.0 - 0 = 2.0 | 300.0 |
| Average | 1397.0 / 3.0 = 465.67 | 3.0 / 3.0 = 1.0 | 400.0 / 3.0 = 133.33 |

### 4. For what types of workloads does SJF deliver the same turnaround
times as FIFO?

SJF delivers the same turnaround times as FIFO when all jobs have the same length. In this case, both schedulers will execute the jobs in the same order, resulting in the same turnaround times.

### 5. For what types of workloads and quantum lengths does SJF deliver
the same response times as RR?

All jobs have the same length and The time quantum for RR is set to be equal to the job length In this scenario, both schedulers will start executing each job at the same time, leading to identical response times.

### 6. What happens to response time with SJF as job lengths increase? Can you use the simulator to demonstrate the trend?

With SJF, as job lengths increase, the response time for later jobs also increases. This is because SJF executes jobs in order of their length, so shorter jobs get prioritized over longer ones. Longer jobs have to wait for all preceding shorter jobs to complete before they can start executing, leading to higher response times.

### 7. What happens to response time with RR as quantum lengths increase? Can you write an equation that gives the worst-case response time, given N jobs?

With RR, as the time quantum (i.e., the time slice) increases, the worst-case response time also increases. This is because a job may have to wait for all other jobs to complete their time slices before it can start executing.
The worst-case response time for RR with N jobs and a time quantum of q can be expressed as:
Worst-case response time = (N - 1) * q
