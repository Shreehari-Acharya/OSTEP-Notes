## Introduction:

- When there is enough free memory, page faults are easy to handle by assigning a free page.
- When memory is limited, the OS needs to decide which page(s) to evict to make room for actively used pages. This decision is encapsulated in the replacement policy.
- Choosing the right page(s) to evict was an important decision in older systems with limited memory.

## Cache Management:

- Main memory can be viewed as a cache for virtual memory pages in the system.
- The goal is to minimize the number of cache misses (or maximize cache hits) to reduce the average memory access time (AMAT).
- AMAT is calculated as: AMAT = T_M + (P_Miss * T_D), where T_M is the cost of accessing memory, T_D is the cost of accessing disk, and P_Miss is the probability of a cache miss.

## The Optimal Replacement Policy:

- The optimal replacement policy, developed by Belady, replaces the page that will be accessed furthest in the future, resulting in the fewest misses.
- Although not practical to implement, it serves as a useful comparison point for other policies.

## Simple Policies:

- FIFO (First-In-First-Out): Replaces the page that was brought into memory first.
- Random: Selects a random page to replace.
- These simple policies perform poorly as they do not consider the importance of pages.

## Using History: LRU (Least Recently Used):

- LRU replaces the least recently used page, based on the principle of locality.
- Locality states that programs tend to access certain code sequences and data structures frequently.
- LRU performs better than simple policies but requires tracking the usage history of pages.

## Workload Examples:

- No-Locality Workload: All policies perform equally when there is no locality.
- 80-20 Workload: LRU performs better than Random and FIFO as it retains the "hot" pages.
- Looping Sequential Workload: LRU and FIFO perform poorly, while Random does better.

## Implementing Historical Algorithms:

- Implementing perfect LRU requires updating data structures on every memory reference, which is expensive.
- An approximation using the use bit (reference bit) is more feasible.

## Approximating LRU:

- The Clock algorithm uses the use bit to approximate LRU.
- It scans pages in a circular list and replaces the first page with a cleared use bit.

## Considering Dirty Pages:

- A modified bit (dirty bit) indicates whether a page has been modified in memory.
- The Clock algorithm can be modified to prefer evicting clean pages over dirty pages to reduce disk writes.

## Other VM Policies:

- Page selection policy (demand paging, prefetching)
- Write policies (clustering/grouping writes)

## Thrashing:

- When memory demands exceed available physical memory, the system enters a state of thrashing (constant paging).
- Techniques like admission control (running a subset of processes) or out-of-memory killer (killing memory-intensive processes) can be used to alleviate thrashing.

## Summary:

- Modern systems use variations of LRU approximations like Clock and ARC.
- With the advent of faster storage devices (e.g., SSDs), page replacement algorithms have regained importance.

The chapter covers various page replacement policies, their implementations, workload examples, and related VM policies like write policies and handling thrashing. It emphasizes the importance of considering history and locality in replacement algorithms to improve performance.

## Questions & Answers:

1. **Generate random addresses with the following arguments: -s 0 -n 10, -s 1 -n 10, and -s 2 -n 10. Change the policy from FIFO, to LRU, to OPT. Compute whether each access in said address traces are hits or misses.**
2. **For a cache of size 5, generate worst-case address reference streams for each of the following policies: FIFO, LRU, and MRU (worst-case reference streams cause the most misses possible. For the worst case reference streams, how much bigger of a cache is needed to improve performance dramatically and approach OPT?**
    - FIFO worst case: Reference a stream of 6 distinct pages, then repeat the sequence. This will cause a miss every 6th reference.
    - LRU worst case: Reference 5 distinct pages, then repeat the sequence in the opposite order. This will cause a miss every reference after the first 5.
    - MRU worst case: Same as LRU worst case, but access the pages in the same order repeatedly.
    To approach OPT performance, a cache size of at least 6 is needed for these worst-case streams.
3. **Generate a random trace (use python or perl). How would you expect the different policies to perform on such a trace?**
    
    For a random trace, all policies are expected to perform similarly, with the hit rate determined primarily by the cache size relative to the working set size of the trace.
    
4. **Now generate a trace with some locality. How can you generate such a trace? How does LRU perform on it? How much better than RAND is LRU? How does CLOCK do? How about CLOCK with different numbers of clock bits?**
    
    To generate a trace with locality, we can use 80-20 distribution LRU is expected to perform better than Random on such a trace, as it will retain the hot pages in the cache. Clock should perform similarly to LRU, and using more clock bits should improve its accuracy in approximating LRU.
    
5. **Use a program like valgrind to instrument a real application and generate a virtual page reference stream. For example, running `valgrind --tool=lackey --trace-mem=yes ls` will output a nearly-complete reference trace of every instruction and data reference made by the program ls. To make this useful for the simulator above, you’ll have to first transform each virtual memory reference into a virtual page-number reference (done by masking off the offset and shifting the resulting bits downward). How big of a cache is needed for your application trace in order to satisfy a large fraction of requests? Plot a graph of its working set as the size of the cache increases.**
