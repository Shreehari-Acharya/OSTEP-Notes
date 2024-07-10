## Flash Basics

### Flash Chip Organization

- Flash chips organized into banks/planes
- Banks contain erase blocks (128KB-256KB)
- Blocks contain pages (few KB each)

### Basic Flash Operations

- Read: Can read one or more pages
- Erase: Erases entire block (sets all bits to 1)
- Program: Changes some 1's to 0's to write data to a page
- Pages have states: INVALID, ERASED, VALID

### Flash Characteristics

- Random access device (like DRAM)
- Asymmetric read/write performance
- Wear out after many erase/program cycles
- Single-level cell (SLC) vs multi-level cell (MLC) vs triple-level cell (TLC)

## Solid State Drives (SSDs)

### Flash Translation Layer (FTL)

- Presents block interface to client
- Translates client reads/writes to flash operations
- Most FTLs are log-structured

### FTL Design Challenges

- Garbage collection
- Mapping table size
- Wear leveling

### FTL Mapping Approaches

- Page-level mapping
- Block-level mapping
- Hybrid mapping
- Page mapping with caching

### Garbage Collection

- Reclaims blocks with invalid/dead pages
- Increases write amplification
- Can use overprovisioning to reduce impact

### Wear Leveling

- Spreads writes evenly across blocks
- Periodically moves static data

## SSD Performance and Cost

### Performance Comparison to HDDs

- Much better random I/O performance
- Comparable sequential performance
- Random writes often faster than random reads due to log structure

### Cost

- SSDs still ~10x more expensive per GB than HDDs
- Hybrid systems use SSDs for hot data, HDDs for cold data

## Key SSD Terms

- Flash chip, bank, erase block, page
- Read, erase, program operations
- Trim operation
- Flash translation layer (FTL)
- Garbage collection
- Write amplification
- Wear leveling

This covers the key points on flash technology, SSD design and implementation, and performance characteristics compared to hard disk drives.
