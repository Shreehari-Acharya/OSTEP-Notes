## Overview of NFS

### Key Goals and Design Principles

- Provide transparent, distributed file access across a network
- Enable simple and fast server crash recovery
- Use a stateless protocol design
- Make operations idempotent to allow safe client retries

### Basic Architecture

- Client-side file system implements NFS protocol
- Server (file server) responds to NFS protocol requests
- Clients cache data and metadata for performance
- Server exports one or more file systems to clients

## NFS Protocol Details

### File Handles

- Used to uniquely identify files/directories
- Contains volume ID, inode number, generation number

### Key Protocol Messages

- LOOKUP - get file handle for a file/directory
- READ - read data from file
- WRITE - write data to file
- GETATTR - get file attributes
- CREATE, REMOVE, etc. for other operations

### Stateless Design

- Server does not keep track of client state
- Each request contains all needed information
- Enables simple crash recovery - clients just retry requests

## Client-side Caching

### Benefits

- Improves performance by avoiding network requests
- Enables write buffering

### Cache Consistency Challenges

- Update visibility - when do updates become visible to other clients?
- Stale cache - how to detect when cached data is out of date?

### NFS Solutions

- Flush-on-close semantics
- Client checks file attributes before using cached data
- Attribute caching to reduce server load

## Server Considerations

### Write Handling

- Must commit writes to stable storage before acknowledging
- Prevents data loss/corruption on server crash
- Can be a performance bottleneck

### Caching

- Can cache reads in memory
- Must be careful with write caching to avoid data loss

## Summary

- Stateless protocol design enables simple crash recovery
- Caching improves performance but introduces consistency challenges
- Engineering tradeoffs made in cache consistency approach
- Server must carefully handle writes for correctness

The NFS design showed how to build a practical, widely-used distributed file system, though with some limitations in its consistency model.
