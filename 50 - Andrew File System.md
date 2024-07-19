## AFS Overview

### Background

- Developed at Carnegie Mellon University in the 1980s
- Led by Professor M. Satyanarayanan ("Satya")
- Main goal was scalability - supporting as many clients per server as possible

### Key Design Principles

- Whole-file caching on client local disk
- Callbacks from server to client for cache invalidation
- Simplified consistency model

## AFS Version 1 (AFSv1)

### Key Features

- Whole-file caching on client local disk
- Client-side caching of file contents only (not directories)
- TestAuth protocol to check if cached file is valid

### Protocol Messages

- Fetch: Get entire file from server
- Store: Send entire file to server
- TestAuth: Check if cached file is still valid

### Limitations

- High path traversal costs on server
- Too many TestAuth messages from clients

## AFS Version 2 (AFSv2)

### Key Improvements

- Callbacks - server promises to notify client if file changes
- File identifiers (FIDs) instead of full pathnames
- Client-side caching of directories

### Protocol Changes

- Fetch now returns file/directory contents and sets up callback
- No need for TestAuth - assume cached copy valid until callback

### Cache Consistency

- Updates visible on server when file closed
- Server breaks callbacks on other clients when file updated
- "Last writer wins" for simultaneous updates

### Crash Recovery

- Clients must revalidate cache contents after reboot
- Server must notify all clients after crash to invalidate caches

## Performance Comparison to NFS

### Advantages of AFS

- Better performance for large file re-reads (from local disk cache)
- Simpler consistency model
- Reduced server load for common operations

### Disadvantages of AFS

- Worse performance for small reads/writes to large files
- Higher overhead for file overwrites

## Other AFS Improvements

- Global namespace
- Better security and access controls
- Improved manageability for administrators
