## **Multi-Level Feedback Queue (MLFQ) Scheduler**

MLFQ is a scheduling algorithm that aims to optimize for two goals: minimizing turnaround time for short jobs and minimizing response time for interactive jobs, without a priori knowledge of job lengths. It works by using multiple distinct queues, each with a different priority level. Jobs are scheduled from the highest priority queue first using round-robin scheduling.

### **Key Rules**

1. **Rule 1:** If priority(A) > priority(B), A runs (B doesn't)
2. **Rule 2:** If priority(A) = priority(B), A & B run in round-robin
3. **Rule 3:** When a job enters the system, it is placed in the highest priority queue
4. **Rule 4:** Once a job uses up its time allotment at a level, its priority is reduced (moves down one queue)
5. **Rule 5:** After some time S, move all jobs to the topmost queue (priority boost)

### **How MLFQ Works**

- MLFQ observes how jobs run over time and prioritizes them accordingly, approximating Shortest Job First (SJF) for short jobs and fairness for long-running CPU-bound jobs.
- Priority boosting (Rule 5) avoids starvation and accounts for changing behavior of jobs.
- Better accounting (Rule 4) prevents gaming where jobs relinquish CPU before time slice ends to remain in a higher priority queue.

### **Examples**

- A single long-running job starts at the highest priority queue and gradually moves down queues as it uses up its time allotment at each level.
- When a short interactive job arrives while a long job is running, MLFQ gives the short job high priority, allowing it to run quickly and complete before the long job.
- If a job does I/O before using up its time allotment, the old rules 4a and 4b kept it at the same priority level. The new rule 4 does better accounting and reduces its priority regardless.

### **Tuning and Other Issues**

- Parameters like the number of queues, time slice lengths, allotment values, and priority boost interval (S) need to be tuned based on the workload.
- Higher queues usually have shorter time slices (10ms) for interactive jobs. Lower queues have longer time slices (100s of ms) for CPU-bound jobs.
- Some systems reserve the highest priority for OS tasks. User advice like "nice" can increase/decrease a job's priority.
- Different algorithms are used in practice, like the decay-usage formulas in FreeBSD instead of strict rules.

### **Preventing Gaming**

- Old rules 4a/4b allowed jobs to game by doing minor I/O to reset their time allotment.
- New rule 4 does better accounting - uses up the full allotment across I/O periods before demotion.

## Homework Questions and their Answers:

**1. Run a few randomly-generated problems with just two jobs and two queues; compute the MLFQ execution trace for each. Make your life easier by limiting the length of each job and turning off I/Os.**

```bash
User@Linux:~/ostep-homework/cpu-sched-mlfq$ python3 ./mlfq.py -j 2 -n 2 -m10 -M 0 -s 25
Here is the list of inputs:
OPTIONS jobs 2
OPTIONS queues 2
OPTIONS allotments for queue  1 is   1
OPTIONS quantum length for queue  1 is  10
OPTIONS allotments for queue  0 is   1
OPTIONS quantum length for queue  0 is  10
OPTIONS boost 0
OPTIONS ioTime 5
OPTIONS stayAfterIO False
OPTIONS iobump False

For each job, three defining characteristics are given:
  startTime : at what time does the job enter the system
  runTime   : the total CPU time needed by the job to finish
  ioFreq    : every ioFreq time units, the job issues an I/O
              (the I/O takes ioTime units to complete)

Job List:
  Job  0: startTime   0 - runTime   4 - ioFreq   0
  Job  1: startTime   0 - runTime   8 - ioFreq   0

Compute the execution trace for the given workloads.
If you would like, also compute the response and turnaround
times for each of the jobs.

Use the -c flag to get the exact results when you are finished.
```

The Job 0 will run completely and then the Job 1

**2. How would you run the scheduler to reproduce each of the exam-
ples in the chapter?**

Example 1: A Single Long-Running Job

```bash
User@Linux:~/ostep-homework/cpu-sched-mlfq$ python3 ./mlfq.py -n 3 -q 10 -j 1 -m 300 -M 0
```

Example 2: Along Came A Short Job

```bash
User@Linux:~/ostep-homework/cpu-sched-mlfq$ python3 ./mlfq.py -n 3 -q 10 -l 0,1000,0:100,20,0
```

Example 3: What About I/O?

```bash
User@Linux:~/ostep-homework/cpu-sched-mlfq$ python3 ./mlfq.py -n 3 -q 10 -l 0,1000,0:50,100,25 -i 9
```

**3. How would you configure the scheduler parameters to behave just
like a round-robin scheduler?**

By specifying the number of queue to be 1

```bash
User@Linux:~/ostep-homework/cpu-sched-mlfq$ python3 ./mlfq.py -n 1
```

**4. Craft a workload with two jobs and scheduler parameters so that one job takes advantage of the older Rules 4a and 4b (turned on with the -S flag) to game the scheduler and obtain 99% of the CPU over a particular time interval.**

```bash
User@Linux:~/ostep-homework/cpu-sched-mlfq$ ./mlfq.py --jlist 0,100,99:0,1000,0 -n 3 -q 10 -a 1 -S
```

**5. Given a system with a quantum length of 10 ms in its highest queue, how often would you have to boost jobs back to the highest priority level (with the -B flag) in order to guarantee that a single long-running (and potentially-starving) job gets at least 5% of the CPU?**

We want the long-running job to get at least 5% of the CPU time. With a quantum of 10 ms in the highest queue, the long-running job will get 10 ms of CPU time per boost interval.

To get 5% of the CPU time, the job needs to run for 10 ms out of every 200 ms (5% of 200 ms = 10 ms).

Therefore, we need to boost the priorities every 200 ms to ensure that the long-running job gets at least 5% of the CPU time.

The command to run the scheduler with this configuration would be:

```bash
User@Linux:~/ostep-homework/cpu-sched-mlfq$ ./mlfq.py -B 200 -q 10
```

This sets the boost interval to 200 ms (`-B 200`) and the quantum length for the highest queue to 10 ms (`-q 10`).

With this configuration, the long-running job will be moved to the highest priority queue every 200 ms and will be able to run for 10 ms (its entire quantum) before being demoted again. This guarantees that the job gets at least 5% of the CPU time.
