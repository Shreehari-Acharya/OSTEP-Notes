## Files and Directories:

- File: A linear array of bytes with a low-level name (inode number)
- Directory: A collection of (name, inode number) pairs, mapping human-readable names to inodes
- Directory tree/hierarchy organizes all files and directories starting from the root directory

### File System Interface:

- Open a file with open() system call to get a file descriptor
- Read/write files with read()/write() system calls using file descriptors
- Use lseek() to change the current offset for random access
- fsync() to force writes to persistent storage
- Create files with open() and O_CREAT flag
- Unlink/delete files with unlink() system call

### Directories:

- Create directories with mkdir() system call
- Read directories with opendir(), readdir(), closedir()
- Remove empty directories with rmdir()

### Links:

- Hard links create additional directory entries pointing to the same inode
- Symbolic (soft) links are special files containing a pathname
- Hard links share the same inode, symbolic links point to a pathname

### File Metadata and Permissions:

- Use stat()/fstat() to get file metadata like size, inode, permissions, etc.
- Permissions control read, write, execute access for owner, group, others
- Use chmod() to change permissions

### Mounting and File Systems:

- mkfs utility creates a new file system on a device
- mount attaches a file system at a mount point in the directory tree
- mount unifies multiple file systems into a single directory hierarchy
