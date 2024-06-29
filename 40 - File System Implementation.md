### The Way To Think

- To understand a file system, think about two aspects:
1. **Data structures**: The on-disk structures utilized by the file system to organize its data and metadata.
2. **Access methods**: How the file system maps system calls (like open(), read(), write(), etc.) onto its structures.

### Overall Organization

- The disk is divided into blocks of a fixed size (e.g., 4 KB).
- The data region stores user data in data blocks.
- The inode table stores metadata about files in inode structures.
- Bitmaps are used to track free and allocated inodes and data blocks.
- The superblock contains information about the file system (e.g., number of inodes, data blocks, inode table location).

### File Organization: The Inode

- The inode (index node) stores metadata about a file, such as size, permissions, access times, and pointers to data blocks.
- The multi-level index approach is used to support large files:
    - Direct pointers to data blocks.
    - Indirect pointers to blocks containing pointers to data blocks.
    - Double indirect pointers to blocks containing pointers to indirect blocks.
    - Triple indirect pointers to blocks containing pointers to double indirect blocks.

### Directory Organization

- Directories contain a list of (entry name, inode number) pairs.
- Each directory has two special entries: `.` (current directory) and `..` (parent directory).

### Free Space Management

- Bitmaps are used to track free and allocated inodes and data blocks.
- Allocation policies may try to allocate contiguous blocks for better performance.

### Access Paths: Reading and Writing

- Reading a file involves traversing the directory hierarchy, reading inodes and data blocks.
- Writing a file involves allocating new inodes and data blocks, updating bitmaps and inodes.
- File creation is expensive, as it involves allocating inodes, updating directories, and potentially allocating new directory blocks.

### Caching and Buffering

- Caching and buffering are used to reduce the I/O costs of reading and writing files.
- A unified page cache integrates virtual memory and file system pages, allowing flexible allocation of memory between them.
- Write buffering delays writes to disk, enabling batching, scheduling, and avoiding unnecessary writes.
- Applications can bypass buffering using fsync(), direct I/O, or raw disk access if desired.
