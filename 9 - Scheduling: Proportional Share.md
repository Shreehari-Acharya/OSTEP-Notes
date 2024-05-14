# Scheduling: Proportional Share

## Crux: How to Share the CPU Proportionally?

- Design a scheduler to share the CPU in a proportional manner
- Key mechanisms for achieving proportional sharing
- How effective are the mechanisms?

## Basic Concept: Tickets Represent Your Share

- Tickets represent the share of a resource a process should receive
- Percent of tickets a process has represents its share of the CPU
- Example: Process A has 75 tickets, Process B has 25 tickets
    - A should get 75% of CPU, B should get 25%

## Ticket Mechanisms

- **Ticket Currency**: Users allocate tickets in their own currency, system converts to global currency
- **Ticket Transfer**: A process can temporarily transfer its tickets to another process (e.g., client to server)
- **Ticket Inflation**: A process can temporarily raise or lower its ticket count

## Lottery Scheduling

- Hold a lottery periodically (e.g., every time slice)
- Pick a random winning ticket between 0 and (total tickets - 1)
- The process holding the winning ticket gets to run

## Implementation

- Need a random number generator, data structure for processes, and total ticket count
- Pick a random winning ticket from 0 to (total tickets - 1)
- Traverse process list, accumulating tickets until sum exceeds winning ticket
- That process is the winner and gets to run

## Stride Scheduling

- Deterministic fair-share scheduler
- Each job has a stride inversely proportional to its ticket count
- Pick job with minimum pass value to run
- When a job runs, increment its pass value by its stride
- Achieves perfect fairness over time without randomness

## Linux Completely Fair Scheduler (CFS)

- Modern approach to fair-share scheduling in Linux
- Focuses on efficiency and scalability

### Basic Operation

- Uses virtual runtime (vruntime) instead of fixed time slices
- Processes accrue vruntime as they run
- Pick process with minimum vruntime to run next
- Uses sched_latency to determine time slice size
- Limits minimum time slice to min_granularity for efficiency

### Weighting (Niceness)

- Uses nice values (-20 to +19) to weight processes
- Calculates time slice and vruntime accumulation based on weight

### Using Red-Black Trees

- Stores runnable processes in a red-black tree ordered by vruntime
- Enables efficient (O(log n)) insertion, deletion, and lookup

### Dealing with I/O and Sleeping Processes

- Sets vruntime of waking process to minimum in tree to avoid starvation
- But processes that sleep frequently may not get fair share

## Summary

- Lottery and stride schedulers provide probabilistic and deterministic fair sharing
- CFS is a widely used, scalable fair-share scheduler in Linux
- Fair-share schedulers have issues with I/O and priority assignment

The key points covered include the basic concept of using tickets to represent resource shares, the lottery and stride scheduling algorithms, implementation details, fairness analysis, and the design of the Linux Completely Fair Scheduler (CFS).
