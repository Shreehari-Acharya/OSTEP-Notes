1. Introduction:
    - Importance of I/O in computer systems
    - Challenge: Efficient integration of I/O into systems
2. System Architecture:
    - Components: CPU, memory, I/O bus, peripheral bus
    - Hierarchical structure due to physics and cost
    - Modern systems: specialized chipsets and point-to-point interconnects
3. Canonical Device:
    - Hardware interface (registers) and internal structure
    - Interface: status, command, and data registers
    - Protocol: poll status, write data, write command, poll status for completion
4. Inefficiencies in Canonical Protocol:
    - Problem: Polling wastes CPU time
    - Solution: Interrupts
        - Allow CPU to switch tasks while waiting for I/O
        - Enable overlap of computation and I/O
    - Note: Interrupts not always better (fast devices, interrupt floods)
5. Programmed I/O (PIO) Inefficiency:
    - Problem: CPU wastes time on data transfer
    - Solution: Direct Memory Access (DMA)
        - Offloads data transfer from CPU
        - DMA engine handles transfers, interrupts CPU when done
6. Methods of Device Interaction:
    - Explicit I/O instructions (e.g., in/out on x86)
    - Memory-mapped I/O: device registers as memory locations
    - Both methods in use today
7. Device Drivers:
    - Problem: Keeping OS device-neutral
    - Solution: Device drivers encapsulate device-specific details
    - Example: Linux file system stack
    - Downside: Generic interfaces may not use all device capabilities
    - Note: Device drivers are a large portion of OS code (70%+ in Linux)
8. Case Study: IDE Disk Driver
    - IDE disk interface: control, command block, status, error registers
    - Basic protocol: wait for readiness, write parameters, start I/O, handle data/interrupts
    - xv6 IDE driver functions:
        - ide_rw(): queue or issue request
        - ide_start_request(): send request to disk
        - ide_wait_ready(): ensure drive readiness
        - ide_intr(): handle interrupts
9. Historical Notes:
    - Interrupts and DMA introduced in early 1950s
    - Exact origins debated (UNIVAC, DYSEAC, IBM SAGE)
    - Ideas arose from need to handle slow I/O devices with fast CPUs
10. Summary:
    - Interrupts and DMA improve device efficiency
    - Device access: I/O instructions or memory-mapped I/O
    - Device drivers make OS device-neutral
