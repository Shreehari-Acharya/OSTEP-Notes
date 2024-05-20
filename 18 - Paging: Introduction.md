# Paging: Introduction

## Overview

- Paging is an approach to virtualize memory by dividing the address space into fixed-sized units called pages.
- Physical memory is divided into fixed-sized slots called page frames, each capable of holding one virtual page.
- Paging avoids external fragmentation by using fixed-size chunks and provides flexibility by supporting sparse address spaces.

## Where Are Page Tables Stored?

- Page tables can be huge (e.g., 4MB for a 32-bit address space with 4KB pages).
- Page tables are stored in physical memory managed by the OS, not in special hardware.

## Page Table Contents

- Page tables use a linear structure (array) indexed by the VPN.
- Each page table entry (PTE) contains:
    - Valid bit: Indicates if the translation is valid.
    - Protection bits: Controls read/write/execute permissions.
    - Present bit: Indicates if the page is in physical memory or on disk.
    - Dirty bit: Indicates if the page has been modified.
    - Accessed/Reference bit: Tracks if the page has been accessed.
    - PFN: The physical frame number where the page resides.

## Paging is Slow

- To access memory, the hardware must first fetch the PTE from the page table, perform the translation, and then access the data in physical memory.
- This extra memory reference for the page table lookup slows down the process by a factor of 2 or more.

## Summary

- Paging avoids external fragmentation and supports sparse address spaces but can lead to slower performance and memory waste due to large page tables.
- Careful design of hardware and software is required to make paging work efficiently.

## Questions & Answers

1. **Before doing any translations, let’s use the simulator to study how linear page tables change size given different parameters. Compute the size of linear page tables as different parameters change. Some suggested inputs are below; by using the -v flag, you can see how many page-table entries are filled. First, to understand how linear page table size changes as the address space grows, run with these flags:**
`-P 1k -a 1m -p 512m -v -n 0`
`-P 1k -a 2m -p 512m -v -n 0`
`-P 1k -a 4m -p 512m -v -n 0`

**Then, to understand how linear page table size changes as page size
grows:**

`-P 1k -a 1m -p 512m -v -n 0`
`-P 2k -a 1m -p 512m -v -n 0`
`-P 4k -a 1m -p 512m -v -n 0`

**Before running any of these, try to think about the expected trends. How should page-table size change as the address space grows? As the page size grows? Why not use big pages in general?**

As the address space size grows:

- Page table size increases linearly with the address space size when the page size is fixed.
- This is because the page table needs one entry for every virtual page in the address space.
- For example, if the page size is 1KB and the address space grows from 1MB to 2MB to 4MB, the number of entries in the page table will double from 1024 to 2048 to 4096.

As the page size grows:

- Page table size decreases as the page size increases when the address space size is fixed.
- This is because with larger pages, fewer entries are needed to map the same address space.
- For example, if the address space is 1MB, with 1KB pages we need 1024 entries, but with 2KB pages we only need 512 entries, and with 4KB pages we only need 256 entries.

In general, we don't use very large page sizes because:

- Larger pages lead to more internal fragmentation (unused part of the last page).
- Larger pages make it harder to share memory between processes efficiently.
- Larger pages require more data to be transferred on page faults, increasing overhead.

  
2. **Now let’s do some translations. Start with some small examples,
and change the number of pages that are allocated to the address
space with the -u flag. For example:**

`-P 1k -a 16k -p 32k -v -u 0`
`-P 1k -a 16k -p 32k -v -u 25`
`-P 1k -a 16k -p 32k -v -u 50`
`-P 1k -a 16k -p 32k -v -u 75`
`-P 1k -a 16k -p 32k -v -u 100`

**What happens as you increase the percentage of pages that are allocated in each address space?**

As you increase the percentage of pages allocated to the address space:

- The number of valid entries in the page table increases.
- More virtual pages are mapped to physical frames.
- With a very low percentage (e.g., 0%), the page table only has invalid entries.
- With a very high percentage (e.g., 100%), most or all virtual pages are mapped.

3. **Now let’s try some different random seeds, and some different (and sometimes quite crazy) address-space parameters, for variety:**

`-P 8 -a 32 -p 1024 -v -s 1`
`-P 8k -a 32k -p 1m -v -s 2`
`-P 1m -a 256m -p 512m -v -s 3`
**Which of these parameter combinations are unrealistic? Why?**

The `-P 8 -a 32 -p 1024` combination is unrealistic because the address space (32 bytes) and physical memory (1024 bytes) sizes are extremely small, and the page size (8 bytes) is quite large relative to them.

4. **Use the program to try out some other problems. Can you find the limits of where the program doesn’t work anymore? For example, what happens if the address-space size is bigger than physical memory?**
- If the address space size is larger than physical memory, the program will report an error because it assumes that physical memory must be larger than the address space for the simulation to work correctly.
- For extremely large address spaces or physical memory sizes (e.g., greater than 1GB), the program will report an error and ask to use smaller sizes, as it is designed for smaller examples.
- The program assumes that the address space size and physical memory size are powers of 2 and multiples of the page size. If these conditions are not met, it will report an error.
