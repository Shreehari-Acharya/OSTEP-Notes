**Virtual Memory and Swap Space**

- The OS needs to support large virtual address spaces for processes even when the physical memory is limited.
- To achieve this, the OS uses a slower, larger storage device (usually a hard disk) as swap space to store portions of address spaces not currently in demand.
- This allows the OS to provide the illusion of a large virtual memory that exceeds the available physical memory.

**Swap Space**

- The OS reserves a space on the disk called swap space to move pages back and forth.
- The size of the swap space determines the maximum number of memory pages that can be in use.
- Pages of different processes can reside partly in physical memory and partly in the swap space on disk.

**The Present Bit**

- Each page table entry has a present bit to indicate if the page is present in physical memory or not.
- If the present bit is 0, it means the page is not in memory but on disk in the swap space.
- Accessing a page not present in memory causes a page fault.

**The Page Fault**

- On a page fault, the OS page-fault handler is invoked to service the fault.
- The handler looks up the disk address of the page in the page table entry and issues a disk I/O to fetch the page from swap space.
- After the I/O completes, the handler updates the page table, marks the page as present, and retries the instruction that caused the fault.

**Page Replacement**

- If physical memory is full when a new page needs to be brought in, the OS needs to replace (evict) one or more existing pages to make room.
- The policy for selecting the page(s) to replace is called the page replacement policy and is critical for good performance.

**High and Low Watermarks**

- To keep some memory free, the OS uses high and low watermark values.
- When free memory falls below the low watermark, a background thread (swap daemon) starts evicting pages until the high watermark is reached.
- The background thread can cluster/group page writes to disk for efficiency.

**Performance Impact**

- Accessing pages in memory is much faster than accessing from disk due to swapping.
- Swapping performance can be improved by using faster storage devices like SSDs or RAID arrays, but it still lags behind in-memory performance.

## Questions & Answers:

1. **First, open two separate terminal connections to the same machine, so that you can easily run something in one window and the other. Now, in one window, run `vmstat 1`, which shows statistics about machine usage every second. Read the man page, the associated README, and any other information you need so that you can understand its output. Leave this window running vmstat for the rest of the exercises below
Now, we will run the program mem.c but with very little memory usage. This can be accomplished by typing `./mem 1` (which uses only 1 MB of memory). How do the CPU usage statistics change when running mem? Do the numbers in the user time column make sense? How does this change when running more than one instance of mem at once?**
    
    The numbers in the user column makes sense. It increases as we run more number of instances of mem.
    
2. **Let’s now start looking at some of the memory statistics while running mem. We’ll focus on two columns: swpd (the amount of virtual memory used) and free (the amount of idle memory). Run `./mem 1024` (which allocates 1024 MB) and watch how these values change. Then kill the running program (by typing control-c) and watch again how the values change. What do you notice about the values? In particular, how does the free column change when the program exits? Does the amount of free memory increase by the expected amount when mem exits?**
    
    As soon as we run `./mem 1024` we can see that the free column decreases. And when we kill the running program the free space is again increased.
    
3. **We’ll next look at the swap columns (si and so), which indicate how much swapping is taking place to and from the disk. Of course, to activate these, you’ll need to run mem with large amounts of memory. First, examine how much free memory is on your Linux system (for example, by typing `cat /proc/meminfo` type `man proc` for details on the `/proc` file system and the types of information you can find there). One of the first entries in `/proc/meminfo` is the total amount of memory in your system. Let’s assume it’s something like 8 GB of memory; if so, start by running mem 4000 (about 4 GB) and watching the swap in/out columns. Do they ever give non-zero values? Then, try with 5000, 6000, etc. What happens to these values as the program enters the second loop (and beyond), as compared to the first loop? How much data (total) are swapped in and out during the second, third, and subsequent loops? (do the numbers make sense?)**
    
    In the second and subsequent loops, all memory accesses will involve swapping data in and out, so the swap in/out values will be very high.
    
4. **Do the same experiments as above, but now watch the other statistics (such as CPU utilization, and block I/O statistics). How do they change when mem is running?**
    
    When `mem` is running, especially with large memory sizes, you'll see an increase in CPU utilization, disk I/O activity (for swapping), and a drop in free memory.
    
5. **Now let’s examine performance. Pick an input for mem that comfortably fits in memory (say 4000 if the amount of memory on the system is 8 GB). How long does loop 0 take (and subsequent loops 1, 2, etc.)? Now pick a size comfortably beyond the size of memory (say 12000 again assuming 8 GB of memory). How long do the loops take here? How do the bandwidth numbers compare? How different is performance when constantly swapping versus fitting everything comfortably in memory? Can you make a graph, with the size of memory used by mem on the x-axis, and the bandwidth of accessing said memory on the y-axis? Finally, how does the performance of the first loop compare to that of subsequent loops, for both the case where everything fits in memory and where it doesn’t?**
    
    the loops should be very fast as they are accessing memory directly. With a larger size like 12000 that requires swapping, the loops will be orders of magnitude slower due to disk I/O. 
    
    The first loop may be slower than subsequent loops for larger sizes due to initial swapping overhead.
    
6. **Swap space isn’t infinite. You can use the tool swapon with the -s flag to see how much swap space is available. What happens if you try to run mem with increasingly large values, beyond what seems to be available in swap? At what point does the memory allocation fail?**
    
    If you try to allocate memory beyond the available swap space, the allocation will fail with an out-of-memory error.
    
7. **Finally, if you’re advanced, you can configure your system to use different swap devices using swapon and swapoff. Read the man pages for details. If you have access to different hardware, see how the performance of swapping changes when swapping to a classic hard drive, a flash-based SSD, and even a RAID array. How much can swapping performance be improved via newer devices? How close can you get to in-memory performance?**
    
    Using faster storage devices like SSDs or RAID arrays for swap space can improve swapping performance significantly compared to a traditional hard disk, but it will still be slower than accessing memory directly.
