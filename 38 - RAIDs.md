## RAID Overview

- RAID (Redundant Array of Inexpensive Disks) is a technique to use multiple disks in concert to build a faster, bigger, and more reliable disk system.
- RAIDs offer advantages like performance, capacity, and reliability over a single disk.
- RAIDs appear as a single disk to the host system, making them transparent and easy to deploy.

## RAID Levels

### RAID Level 0: Striping

- Data is striped across multiple disks in a round-robin fashion.
- Provides high performance but no redundancy.

### RAID Level 1: Mirroring

- Data is mirrored (duplicated) across multiple disks.
- Provides reliability by allowing recovery from disk failures.
- Higher capacity cost due to redundancy.

### RAID Level 4: Parity-Based Redundancy

- Uses parity information for redundancy across a stripe of data blocks.
- Parity is calculated using XOR operation.
- Suffers from the small-write problem due to the parity disk bottleneck.

### RAID Level 5: Rotating Parity

- Similar to RAID-4, but parity block is rotated across disks.
- Addresses the small-write problem of RAID-4 by distributing parity across disks.

## RAID Analysis

- RAIDs are evaluated based on capacity, reliability, and performance.
- Performance analysis considers sequential and random workloads, as well as read and write operations.
- Different RAID levels offer trade-offs between capacity, reliability, and performance.

## Other RAID Issues

- Handling disk failures, reconstruction, and more realistic fault models.
- Software RAID implementations and the consistent-update problem.
- Choosing the right RAID level and parameters for a specific workload.

## Summary

- RAID provides a transparent way to improve capacity, reliability, and performance over a single disk.
- Different RAID levels cater to different priorities and workload requirements.
- Selecting the right RAID level and configuration is crucial for optimal performance and reliability.

## Questions & Answers:

1. Use the simulator to perform some basic RAID mapping tests. Run with different levels (0, 1, 4, 5) and see if you can figure out the mappings of a set of requests. For RAID-5, see if you can figure out the difference between left-symmetric and left-asymmetric layouts. Use some different random seeds to generate different problems than above.
    
    Here is the formula to find mappings of a set of requests for all RAID Levels
    
    ```bash
    disk   = address % number_of_disks
    offset = address / number_of_disks
    ```
    
2. Do the same as the first problem, but this time vary the chunk size with -C. How does chunk size change the mappings?
    
    It makes the mapping smaller.
    
3. Do the same as above, but use the -r flag to reverse the nature of each problem.
    
    Here is the modified formula to find answers for reverse.
    
    ```bash
    Address Block = disk_number + (number_of_disks * offset)
    ```
    
4. Now use the reverse flag but increase the size of each request with the -S flag. Try specifying sizes of 8k, 12k, and 16k, while varying the RAID level. What happens to the underlying I/O pattern when
the size of the request increases? Make sure to try this with the sequential workload too (-W sequential); for what request sizes are RAID-4 and RAID-5 much more I/O efficient?
    
    RAID-4 and RAID-5 are much more I/O efficient for large sequential write requests.
    
5. Use the timing mode of the simulator (-t) to estimate the performance of 100 random reads to the RAID, while varying the RAID levels, using 4 disks.
    
    The timing is from 16 to 22 units.
    
6. Do the same as above, but increase the number of disks. How does the performance of each RAID level scale as the number of disks increases?
    
    For all levels of RAID: As we increase the number of disks, the time taken reduces. Therefore it performs better.
    
7. Do the same as above, but use all writes (-w 100) instead of reads. How does the performance of each RAID level scale now? Can you do a rough estimate of the time it will take to complete the workload of 100 random writes?
    
    Raid 0,1 perform better than 4,5.
    
8. Run the timing mode one last time, but this time with a sequential workload (-W sequential). How does the performance vary with RAID level, and when doing reads versus writes? How about
when varying the size of each request? What size should you write to a RAID when using RAID-4 or RAID-5?
    
    for RAID 4 and RAID 5, you should write requests that are at least as large as the stripe size (chunk size * number of data disks) to obtain the best sequential write performance from full-stripe writes.
