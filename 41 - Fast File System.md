## Problems with the old UNIX file system:

- Poor performance, delivering only 2% of disk bandwidth
- Treated disk like random-access memory, spreading data all over
- Files became fragmented over time
- Small block size (512 bytes) was inefficient for transfers

## FFS solution: Disk awareness

- Designed file system structures and allocation policies to be "disk aware"
- Improved performance while keeping same file system interface/APIs

### Key FFS structures:

- Cylinder groups - divides disk into groups of cylinders
- Each group contains:
    - Superblock copy
    - Inode bitmap
    - Data bitmap
    - Inode region
    - Data block region

### FFS placement policies:

- Keep related items together in same cylinder group
- Place directories in groups with few directories and many free inodes
- Place files in same group as parent directory
- Exception for large files - spread across multiple groups

### Benefits of FFS policies:

- Keeps file data near its inode
- Keeps files in same directory near each other
- Preserves "namespace locality"

### Large file exception:

- After first chunk in same group as inode, spread subsequent chunks across groups
- Prevents single large file from filling a group
- Chunk size chosen to amortize seek costs

### Other FFS innovations:

- Sub-blocks for small files to reduce internal fragmentation
- Optimized disk layout to avoid rotational delays
- Long file names
- Symbolic links
- Atomic rename operation

### Impact:

- Watershed moment in file system history
- Showed importance of treating disk like a disk
- Influenced many subsequent file systems

### Key Concepts:

- Cylinder groups
- Disk-aware placement policies
- Namespace locality
- Amortization of seek costs
- Sub-blocks
- Parameterized disk layout

The chapter emphasizes how FFS dramatically improved performance over the old UNIX file system by optimizing data structures and policies for disk characteristics, while maintaining the same APIs. This disk-aware design influenced many subsequent file systems.
