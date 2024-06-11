## Introduction

- This chapter focuses on understanding the details of hard disk drives, which have been the main form of persistent data storage for decades.
- Understanding disk operation is essential before building file system software that manages disks.

## The Disk Interface

- A disk consists of numbered sectors (512-byte blocks), typically from 0 to n-1.
- Only single-sector writes are guaranteed to be atomic; larger writes may be torn on power loss.
- Unwritten contract assumptions: accessing nearby blocks is faster than distant blocks, and sequential access is faster than random access.

## Basic Geometry

- Disks have one or more platters, each with two surfaces coated with a magnetic layer.
- Platters are bound together around a spindle and rotate at a fixed rate (e.g., 7,200-15,000 RPM).
- Data is stored in concentric circles (tracks) on each surface.
- A disk head attached to a disk arm reads/writes data on each surface.

## A Simple Disk Drive

- Single-track Latency (Rotational Delay): Time to wait for the desired sector to rotate under the disk head.
- Multiple Tracks: Seek Time is the time to move the disk arm to the desired track.
- Seek consists of acceleration, coasting, deceleration, and settling phases.
- I/O Time = Seek Time + Rotational Delay + Transfer Time

## Other Disk Details

- Track skew: Offsetting sectors on adjacent tracks to allow for head switch time.
- Multi-zoned disks: Outer tracks have more sectors than inner tracks due to geometry.
- Disk cache (track buffer): Small memory to cache read/written data for faster subsequent accesses.
- Write policies: Write-back (acknowledge after writing to cache) or write-through (acknowledge after writing to disk).

## Disk Performance Analysis

- Random workload (small, random reads) vs. Sequential workload (large, sequential reads)
- Performance gap between random and sequential workloads can be up to 200-300x.
- High-end "performance" drives are faster but more expensive than low-end "capacity" drives.

## Disk Scheduling

- Disk scheduler determines the order of I/O requests to improve performance.
- SSTF (Shortest Seek Time First) / NBF (Nearest Block First): Schedules the nearest request first, but can lead to starvation.
- Elevator (SCAN, C-SCAN): Sweeps across tracks, servicing requests in order, avoiding starvation.
- SPTF (Shortest Positioning Time First): Accounts for both seek and rotational delays, but difficult to implement in OS.

## Other Scheduling Issues

- Modern disks have internal schedulers and handle multiple outstanding requests from the OS.
- I/O merging: Combining adjacent requests into a single larger request.
- Anticipatory scheduling: Sometimes it's better to wait for a potentially better request before issuing I/O.

## Historical Notes

- Interrupts and DMA ideas emerged in the early 1950s, though the exact origins are debated.
- Disk scheduling algorithms like SCAN and SATF were proposed in the 1970s.

## Summary

- Understanding the functional model of disk drives is essential for building efficient systems.
- Disk scheduling algorithms aim to improve I/O performance by reordering requests.
- Accounting for both seek and rotational delays is important for optimal scheduling.

## Questions & Answers:

1. Compute the seek, rotation, and transfer times for the following sets of requests: `-a 0, -a 6, -a 30, -a 7,30,8,` and finally `-a 10,11,12,13`.
    
    ```bash
    For -a 0
    Block:   0  Seek:  0  Rotate:165  Transfer: 30  Total: 195
    
    TOTALS      Seek:  0  Rotate:165  Transfer: 30  Total: 195
    ------------------------------------------------------------------------
    
    For -a 6
    Block:   6  Seek:  0  Rotate:345  Transfer: 30  Total: 375
    
    TOTALS      Seek:  0  Rotate:345  Transfer: 30  Total: 375
    ------------------------------------------------------------------------
    
    For -a 30
    Block:  30  Seek: 80  Rotate:265  Transfer: 30  Total: 375
    
    TOTALS      Seek: 80  Rotate:265  Transfer: 30  Total: 375
    ------------------------------------------------------------------------
    
    For -a 7,30,8
    Block:   7  Seek:  0  Rotate: 15  Transfer: 30  Total:  45
    Block:  30  Seek: 80  Rotate:220  Transfer: 30  Total: 330
    Block:   8  Seek: 80  Rotate:310  Transfer: 30  Total: 420
    
    TOTALS      Seek:160  Rotate:545  Transfer: 90  Total: 795
    ------------------------------------------------------------------------
    
    For -a 10,11,12,13
    Block:  10  Seek:  0  Rotate:105  Transfer: 30  Total: 135
    Block:  11  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  12  Seek: 40  Rotate:320  Transfer: 30  Total: 390
    Block:  13  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    
    TOTALS      Seek: 40  Rotate:425  Transfer:120  Total: 585
    ------------------------------------------------------------------------
    ```
    
2. Do the same requests above, but change the seek rate to different values: -S 2, -S 4, -S 8, -S 10, -S 40, -S 0.1. How do the times change?
    
    The time remains same, except when -S is 0.1, the time increases significantly.
    
3. Do the same requests above, but change the rotation rate: -R 0.1, -R 0.5, -R 0.01. How do the times change?
    
    The smaller the rotation rate, the higher the rotation rates.
    
4. FIFO is not always best, e.g., with the request stream -a 7,30,8, what order should the requests be processed in? Run the shortest seek-time first (SSTF) scheduler (-p SSTF) on this workload; how long should it take (seek, rotation, transfer) for each request to be served?
    
    ```bash
    FIFO 
    Block:   7  Seek:  0  Rotate: 15  Transfer: 30  Total:  45
    Block:  30  Seek: 80  Rotate:220  Transfer: 30  Total: 330
    Block:   8  Seek: 80  Rotate:310  Transfer: 30  Total: 420
    
    TOTALS      Seek:160  Rotate:545  Transfer: 90  Total: 795
    ---------------------------------------------------------------------------
    
    SSTF
    Block:   7  Seek:  0  Rotate: 15  Transfer: 30  Total:  45
    Block:   8  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  30  Seek: 80  Rotate:190  Transfer: 30  Total: 300
    
    TOTALS      Seek: 80  Rotate:205  Transfer: 90  Total: 375
    ```
    
    As we can see SSTF is much better as it first looks for the blocks which are near to the head. There by accessing the block 7 and 8 before 30.
    
5. Now use the shortest access-time first (SATF) scheduler (-p SATF). Does it make any difference for -a 7,30,8 workload? Find a set of requests where SATF outperforms SSTF; more generally, when is SATF better than SSTF?
    
    ```bash
    SSTF for -a 7,30,8
    Block:   7  Seek:  0  Rotate: 15  Transfer: 30  Total:  45
    Block:   8  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  30  Seek: 80  Rotate:190  Transfer: 30  Total: 300
    
    TOTALS      Seek: 80  Rotate:205  Transfer: 90  Total: 375
    -------------------------------------------------------------------------
    
    SATF for -a 7,30,8
    Block:   7  Seek:  0  Rotate: 15  Transfer: 30  Total:  45
    Block:   8  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  30  Seek: 80  Rotate:190  Transfer: 30  Total: 300
    
    TOTALS      Seek: 80  Rotate:205  Transfer: 90  Total: 375
    ```
    
    We see that there is no difference for workload 7,30,8
    
    I will be using the example from the book. The workload where we can expect different answers for SSTF and SATF is `-a 17,33`
    
    ```bash
    SSTF for -a 17,33
    User@Linux:~/ostep-homework/file-disks$ python3 disk.py -a 17,33 -S 2 -p SSTF -c
    
    Block:  17  Seek: 20  Rotate:295  Transfer: 30  Total: 345
    Block:  33  Seek: 20  Rotate: 70  Transfer: 30  Total: 120
    
    TOTALS      Seek: 40  Rotate:365  Transfer: 60  Total: 465
    -------------------------------------------------------------------------------
    
    SATF for -a 17,33
    User@Linux:~/ostep-homework/file-disks$ python3 disk.py -a 17,33 -S 2 -p SATF -c
    
    Block:  33  Seek: 40  Rotate: 35  Transfer: 30  Total: 105
    Block:  17  Seek: 20  Rotate:190  Transfer: 30  Total: 240
    
    TOTALS      Seek: 60  Rotate:225  Transfer: 60  Total: 345
    ```
    
    “**What it depends on here is the relative time of seeking as compared
    to rotation. If, in our example, seek time is much higher than rotational
    delay, then SSTF (and variants) are just fine. However, imagine if seek is
    quite a bit faster than rotation. Then, in our example, it would make more
    sense to seek further to service request 8 on the outer track than it would
    to perform the shorter seek to the middle track to service 16, which has to
    rotate all the way around before passing under the disk head.**” -OSTEP Book
    
6. Here is a request stream to try: -a 10,11,12,13. What goes poorly when it runs? Try adding track skew to address this problem (-o skew). Given the default seek rate, what should the skew be to maximize performance? What about for different seek rates (e.g., -S 2, -S 4)? In general, could you write a formula to figure out the skew?
    
    skew of 2 gives us the maximum performance, ans skew of 4 does not give us maximum performance.
    
7. Specify a disk with different density per zone, e.g., -z 10,20,30, which specifies the angular difference between blocks on the outer, middle, and inner tracks. Run some random requests (e.g., -a -1 -A 5,-1,0, which specifies that random requests should be used via the -a -1 flag and that five requests ranging from 0 to the max be generated), and compute the seek, rotation, and transfer times. Use different random seeds. What is the bandwidth (in sectors per unit time) on the outer, middle, and inner tracks?
8. A scheduling window determines how many requests the disk can examine at once. Generate random workloads (e.g., -A 1000,-1,0, with different seeds) and see how long the SATF scheduler takes when the scheduling window is changed from 1 up to the number of requests. How big of a window is needed to maximize performance? Hint: use the -c flag and don’t turn
on graphics (-G) to run these quickly. When the scheduling window is set to 1, does it matter which policy you are using?
    
    A Scheduling window of 250 is needed to maximize performance. When scheduling window is set to 1, it does not matter what policy we are using.
    
9. Create a series of requests to starve a particular request, assuming an SATF policy. Given that sequence, how does it perform if you use a bounded SATF (BSATF) scheduling approach? In this approach, you specify the scheduling window (e.g., -w 4); the scheduler only moves onto the next window of requests when all requests in the current window have been serviced. Does this solve starvation? How does it perform, as compared to SATF? In general, how should a disk make this trade-off between performance and starvation avoidance?
    
    `-a 35,8,7,9,10,11,12,13,14,15,16,17,18,19,20,21,22,7,8,10,11` will cause block 35 to starve.
    
    ```bash
    Performance of SATF for -a 35,8,7,9,10,11,12,13,14,15,16,17,18,19,20,21,22,7,8,10,11
    
    Block:   7  Seek:  0  Rotate: 15  Transfer: 30  Total:  45
    Block:   8  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:   9  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  10  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  11  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  14  Seek: 40  Rotate: 20  Transfer: 30  Total:  90
    Block:  15  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  16  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  17  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  18  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  19  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  20  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  21  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  22  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  12  Seek:  0  Rotate: 30  Transfer: 30  Total:  60
    Block:  13  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:   7  Seek: 40  Rotate:110  Transfer: 30  Total: 180
    Block:   8  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  10  Seek:  0  Rotate: 30  Transfer: 30  Total:  60
    Block:  11  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  35  Seek: 80  Rotate:250  Transfer: 30  Total: 360
    
    TOTALS      Seek:160  Rotate:455  Transfer:630  Total:1245
    
    ---------------------------------------------------------------------------
    Performance of BSATF for -a 35,8,7,9,10,11,12,13,14,15,16,17,18,19,20,21,22,7,8,10,11
    
    Block:   7  Seek:  0  Rotate: 15  Transfer: 30  Total:  45
    Block:   8  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:   9  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  35  Seek: 80  Rotate:310  Transfer: 30  Total: 420
    Block:  10  Seek: 80  Rotate:220  Transfer: 30  Total: 330
    Block:  11  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  12  Seek: 40  Rotate:320  Transfer: 30  Total: 390
    Block:  13  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  14  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  15  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  16  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  17  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  18  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  19  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  20  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  21  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  22  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:   7  Seek: 40  Rotate:200  Transfer: 30  Total: 270
    Block:   8  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    Block:  10  Seek:  0  Rotate: 30  Transfer: 30  Total:  60
    Block:  11  Seek:  0  Rotate:  0  Transfer: 30  Total:  30
    
    TOTALS      Seek:240  Rotate:1095  Transfer:630  Total:1965
    
    ```
    
    We can clearly see that although it does not starve block 35 it does make it more time consuming.The choice between SATF and BSATF ultimately depends on the specific requirements of the system and the importance placed on avoiding starvation versus maximizing overall throughput. Adaptive or hybrid approaches that dynamically adjust the scheduling policy or window size based on workload patterns and performance metrics could provide a more balanced trade-off between these competing objectives.
    
10. All the scheduling policies we have looked at thus far are greedy; they pick the next best option instead of looking for an optimal schedule. Can you find a set of requests in which greedy is not optimal?
    
    `-a 6,30,20,10` is one such request where it will perform the worst. Use the -G option to understand it in detail.
