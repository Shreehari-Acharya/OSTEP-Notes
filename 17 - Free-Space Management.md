## Introduction

- Managing free space is a fundamental aspect of memory management systems like malloc libraries or OS memory management.
- The problem is dealing with external fragmentation - when free space gets fragmented into small pieces, making it difficult to satisfy larger requests even when total free space is sufficient.

## Assumptions

- Focusing on malloc/free interface where the library has to track the size of allocated regions when freed.
- Assumes a contiguous region of bytes to manage (the heap).
- No compaction of free space is possible (regions cannot be moved once allocated).

### Low-level Mechanisms

**a. Splitting and Coalescing**

- Splitting: Dividing a free chunk into two when a smaller request needs to be satisfied.
- Coalescing: Merging adjacent free chunks when a region is freed.

**b. Tracking Allocated Region Sizes**

- Use a header before the allocated region to store metadata like size and a magic number.
- On free, the library can find the header and retrieve the size.

**c. Embedding the Free List**

- The free list is built inside the heap memory itself.
- Free chunks are tracked using a linked list node structure embedded in the free space.
- The list is initialized with one node describing the entire heap.
- Operations like splitting and coalescing update the free list nodes.

**d. Growing the Heap**

- If the heap runs out of space, the allocator can request more memory from the OS (e.g., sbrk system call).

### Basic Strategies

**a. Best Fit**

- Search the free list and find the smallest chunk that can accommodate the request.
- Tries to minimize wasted space but has high search overhead.

**b. Worst Fit**

- Find the largest chunk and split it, keeping the remaining large chunk free.
- Performs poorly and leads to excess fragmentation.

**c. First Fit**

- Return the first chunk that fits the request, without searching the entire list.
- Fast but can pollute the beginning of the list with small chunks.

**d. Next Fit**

- Like First Fit, but start searching from the last place looked.
- Helps spread allocations throughout the list.

### Other Approaches

**a. Segregated Lists**

- Maintain separate lists for common request sizes.
- Avoids fragmentation and provides fast allocation/free for those sizes.

**b. Buddy Allocation**

- Divide the heap into powers of two chunks.
- Allocate the smallest buddies that can satisfy the request.
- Coalescing is simple due to the buddy relationship.

**c. Advanced Data Structures**

- Use trees or other complex data structures for better scaling.

**d. Multithreaded Allocators**

- Allocators designed to work well in multithreaded/multiprocessor environments.

## Questions & Answers

1. **First run with the flags -n 10 -H 0 -p BEST -s 0 to generate a few random allocations and frees. Can you predict what alloc()/free() will return? Can you guess the state of the free list after each request? What do you notice about the free list over time?**

When running with these flags, the program will perform 10 random allocation and free operations using the Best Fit policy, without any headers, and with a random seed of 0. The alloc() function will return the starting address of the allocated memory block if successful, or -1 if it fails to find a suitable free block. The free() function will return 0 if successful or -1 if the address is not found in the size map.

It's difficult to predict the exact addresses returned by alloc() or the state of the free list since it depends on the random sequence of operations. However, with the Best Fit policy, the allocator will try to find the smallest free block that can accommodate the requested size, potentially leaving smaller free blocks in the list over time, leading to fragmentation.

2. **How are the results different when using a WORST fit policy to search the free list (-p WORST)? What changes?**

With the Worst Fit policy (-p WORST), the allocator will search for the largest free block that can accommodate the request, instead of the smallest as in Best Fit. This policy tends to leave smaller free blocks in the list, potentially leading to more fragmentation over time compared to Best Fit.

3. **What about when using FIRST fit (-p FIRST)? What speeds up when you use first fit?**

With the First Fit policy (-p FIRST), the allocator will return the first free block it encounters that can satisfy the request, without searching the entire list. This approach is generally faster than Best Fit or Worst Fit because it doesn't need to search the entire list for the best or worst fit. However, it may lead to fragmentation over time, similar to Worst Fit.

4. **For the above questions, how the list is kept ordered can affect the time it takes to find a free location for some of the policies. Use the different free list orderings (-l ADDRSORT, -l SIZESORT+, -l SIZESORT-) to see how the policies and the list orderings interact.**

The order in which the free list is maintained can affect the performance of the allocator, especially for the Best Fit and Worst Fit policies, which require searching the entire list.

- With -l ADDRSORT, the free list is sorted by the starting addresses of the free blocks. This ordering may help the Best Fit policy find the smallest block more quickly if the blocks are generally allocated and freed in address order.
- With -l SIZESORT+, the free list is sorted in ascending order of block sizes. This ordering may help the Best Fit policy find the smallest block more quickly by placing smaller blocks at the start of the list.
- With -l SIZESORT-, the free list is sorted in descending order of block sizes. This ordering may help the Worst Fit policy find the largest block more quickly by placing larger blocks at the start of the list.
5. **Increase the number of random allocations (say to -n 1000). What happens to larger allocation requests over time? Run with and without coalescing (i.e., without and with the -C flag). What differences in outcome do you see? How big is the free list over time in each case? Does the ordering of the list matter in this case?**

Increasing the number of random operations to 1000 (-n 1000) will likely lead to more fragmentation over time, making it harder to satisfy larger allocation requests.

Without coalescing (-C), the free list will contain more fragmented blocks, and larger allocation requests may fail even if the total free space is sufficient. The free list will grow larger over time as more blocks are freed, with many small blocks.

With coalescing (-C), the allocator will attempt to merge adjacent free blocks when a block is freed. This will help reduce fragmentation and make it easier to satisfy larger allocation requests. The free list will be smaller than without coalescing, containing fewer but larger blocks.

In both cases, the ordering of the free list may affect the performance of the allocator, especially for the Best Fit and Worst Fit policies, as discussed in the previous question.

6. **What happens when you change the percent allocated fraction -P to higher than 50? What happens to allocations as it nears 100? What about as the percent nears 0?**

The -P flag controls the probability of performing an allocation versus a free operation.

If you increase the percent allocated fraction (-P) to a value higher than 50, there will be more allocation operations than free operations. This will lead to more memory being allocated over time, potentially causing allocation failures if the heap size is not large enough.

As the percent allocated fraction approaches 100, there will be very few free operations, and the heap will become increasingly full, making it harder to satisfy allocation requests, especially larger ones.

On the other hand, if the percent allocated fraction approaches 0, there will be very few allocation operations, and the heap will become increasingly empty, making it easier to satisfy allocation requests of any size.

7. **What kind of specific requests can you make to generate a highly-fragmented free space? Use the -A flag to create fragmented free lists, and see how different policies and options change the organization of the free list.**

To generate a highly fragmented free space, you can use the -A flag to specify a specific sequence of allocation and free operations. For example, you could try a pattern like:

- A "+1,-0,+2,-1,+3,-2,+4,-3,+5,-4"

This sequence will allocate blocks of sizes 1, 2, 3, 4, and 5, and then free them in the reverse order (0, 1, 2, 3, 4), leaving many small free blocks scattered throughout the heap.

With different allocation policies and list orderings, you can observe how the free list is organized and fragmented after this sequence of operations. Some policies and orderings may handle fragmentation better than others, making it easier or harder to satisfy subsequent allocation requests.
