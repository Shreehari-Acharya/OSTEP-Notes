## Data Integrity and Protection

### Disk Failure Modes

- Traditional model: Fail-stop - entire disk either works or fails completely
- Modern disks exhibit more complex partial failure modes:

### Latent Sector Errors (LSEs)

- Occur when a disk sector or group of sectors is damaged
- Disk returns an error when trying to read the sector
- More common in cheaper drives (9.4%) than costly drives (1.4%)
- Number of LSEs increases with disk size and age

### Block Corruption

- Data becomes corrupt in a way not detectable by the disk itself
- Can be caused by firmware bugs, faulty bus transfers, etc.
- Silent faults - disk gives no indication of the problem
- Less common than LSEs - 0.5% in cheap drives, 0.05% in costly drives

### Handling Latent Sector Errors

- Relatively straightforward to handle since they are easily detected
- Use redundancy mechanisms like RAID to reconstruct data from other copies
- RAID-DP adds extra parity to handle LSEs during RAID reconstruction

### Detecting Corruption: Checksums

- Primary mechanism used to preserve data integrity
- Checksum: Result of a function computed over a chunk of data
- Allows detection of data corruption by comparing stored and computed checksums

### Common Checksum Functions

- XOR-based
- Addition
- Fletcher checksum
- Cyclic redundancy check (CRC)

### Checksum Layout

- Basic approach: Store checksum with each disk sector/block
- Challenges with 512-byte sector sizes
- Alternative: Group checksums together in separate sectors

### Using Checksums

- When reading data:
    1. Read stored checksum
    2. Compute checksum over retrieved data
    3. Compare stored and computed checksums
    4. If mismatch, corruption detected
- On detecting corruption:
    - Use redundant copy if available
    - Return error if no copy exists

### Handling Misdirected Writes

- Occurs when data is written to wrong location on disk
- Solution: Add physical identifier (disk number, sector number) to checksum
- Allows detection of writes to incorrect locations

### Handling Lost Writes

- Occurs when device reports write completed but data not actually persisted
- Solutions:
    - Write verify / read-after-write (but slow)
    - Store checksums elsewhere (e.g. in filesystem metadata)

### Scrubbing

- Periodically read all blocks and verify checksums
- Reduces chances of all copies becoming corrupted over time
- Typically scheduled nightly or weekly

### Overheads of Checksumming

### Space Overheads

- On-disk: Extra space for stored checksums (~0.19% for 8-byte checksum per 4KB block)
- In-memory: Space for checksums when accessing data

### Time Overheads

- CPU time to compute checksums on writes and reads
- Potential extra I/O if checksums stored separately
- Background scrubbing activity

## Summary

Data protection in storage systems relies heavily on checksums to detect corruption and other errors. Different checksum schemes protect against different failure modes. As storage technologies evolve, new protection mechanisms may be needed to handle emerging failure patterns.
