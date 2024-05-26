### Introduction

- Covers how complete virtual memory (VM) systems are built by looking at two examples: VAX/VMS and Linux
- Explores key features for performance, functionality, and security

### VAX/VMS Virtual Memory

- Overview of VAX-11 architecture and VMS operating system
- Memory Management Hardware:
    - 32-bit virtual address space per process, divided into 512-byte pages
    - Hybrid of paging and segmentation
    - Process space and system space
    - Page table for each segment (P0 and P1) per process, in kernel virtual memory
    - Multi-level address translation due to page tables in kernel memory
- A Real Address Space:
    - Code segment doesn't start at page 0 (for null pointer detection)
    - Kernel virtual address space mapped into each user address space
    - Kernel portion is shared across processes
- Page Replacement:
    - No reference bit in page table entry (PTE)
    - Segmented FIFO replacement policy
    - Global clean-page and dirty-page lists for second chance
- Optimizations:
    - Demand zeroing of pages
    - Copy-on-write (COW)
    - Clustering for efficient swap I/O

### The Linux Virtual Memory System

- Linux Address Space:
    - Split between user and kernel portions
    - Kernel logical addresses and kernel virtual addresses
    - Direct mapping between kernel logical addresses and physical memory
- Page Table Structure:
    - Multi-level page table structure based on x86 hardware
    - 64-bit virtual addresses with 4-level page tables (48 bits used currently)
- Large Page Support:
    - Support for huge pages (2MB, 1GB) for better TLB performance
    - Incremental adoption: explicit requests initially, then transparent support
- The Page Cache:
    - Unified cache for file data, anonymous memory, and metadata
    - Background writeback of dirty pages
    - 2Q replacement algorithm (approximating LRU)
- Memory Mapping and Security:
    - Buffer overflow attacks and defenses (NX bit, ASLR, KASLR)
    - Meltdown and Spectre attacks, and KPTI defense
