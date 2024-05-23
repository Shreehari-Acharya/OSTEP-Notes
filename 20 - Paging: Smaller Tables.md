# Paging: Smaller Tables

## CRUX: How to Make Page Tables Smaller?

- Simple array-based page tables (linear page tables) are too big, taking up far too much memory.
- We need techniques to reduce the heavy burden of large page tables.

## Simple Solution: Bigger Pages

- Using bigger pages can reduce the size of the page table.
- But bigger pages lead to internal fragmentation (waste within each page).
- Most systems use relatively small page sizes (4KB or 8KB) in the common case.

## Hybrid Approach: Paging and Segments

- Combines paging and segmentation to reduce page table memory overhead.
- Each segment (code, heap, stack) has its own smaller page table.
- Bounds register indicates the maximum valid page in the segment.
- Advantages: Compact for sparsely used address spaces, pages of page table fit in a page.
- Disadvantages: Assumes a certain address space usage pattern, external fragmentation.

## Multi-level Page Tables

- Divide the linear page table into page-sized units (pieces).
- Use a page directory to track which pieces of the page table are allocated.
- Advantages: Allocates page table space proportional to address space usage, pieces fit in pages.
- Disadvantages: Two memory accesses for a TLB miss (time-space trade-off), more complexity.
- Can have more than two levels if the page directory gets too big.

## Inverted Page Tables

- A single page table with an entry for each physical page in the system.
- Entry tells which process is using the page and which virtual page maps to it.
- Uses a hash table for fast lookups.

## Swapping the Page Tables to Disk

- Page tables can be placed in kernel virtual memory and swapped to disk when memory is tight.

## Summary

- Page tables are data structures, and different structures have time-space trade-offs.
- Choose the right structure based on memory constraints and workload characteristics.

## Questions & Answers:

1. **With a linear page table, you need a single register to locate the page table, assuming that hardware does the lookup upon a TLB miss. How many registers do you need to locate a two-level page table? A three-level table?**
    
    With a linear page table, you need a single register (the Page Table Base Register) to locate the page table. With a two-level page table, you need two registers - the Page Directory Base Register and the Page Table Base Register. With a three-level page table, you need three registers - the Top-Level Page Directory Base Register, the Second-Level Page Directory Base Register, and the Page Table Base Register.
    
2. **Use the simulator to perform translations given random seeds 0, 1, and 2, and check your answers using the -c flag. How many memory references are needed to perform each lookup?**
    
    For a two-level page table, two memory references are needed for each lookup upon a TLB miss (one for the page directory entry, one for the page table entry).
    
3. **Given your understanding of how cache memory works, how do you think memory references to the page table will behave in the cache? Will they lead to lots of cache hits (and thus fast accesses?) Or lots of misses (and thus slow accesses)?**
    
    Memory references to the page table are unlikely to lead to many cache hits, because:
    
    - The working set of accesses to the page table is quite large (the entire page table)
    - There is little temporal or spatial locality in accesses to the page table
