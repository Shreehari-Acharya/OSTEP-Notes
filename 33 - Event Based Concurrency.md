1. Introduction:
    - Event-based concurrency is an alternative to thread-based concurrency.
    - Popular in GUI applications, internet servers, and frameworks like node.js.
    - Addresses challenges of managing concurrency in multi-threaded apps and lack of control over scheduling.
2. Basic Idea: Event Loop
    - Wait for events to occur, then process them one at a time.
    - Main loop: `while(1) { events = getEvents(); for (e in events) processEvent(e); }`
    - Event handlers process events sequentially, providing control over scheduling.
3. select() and poll() APIs:
    - Used to determine which events (like I/O) are taking place.
    - `select()`: checks if file descriptors are ready for reading, writing, or have pending conditions.
    - `poll()`: similar to `select()`.
    - These APIs enable non-blocking event loops.
4. Using select():
    - Example code uses `FD_ZERO()`, `FD_SET()`, and `FD_ISSET()` to manage file descriptors.
    - `select()` is used to check which descriptors have data available.
5. Benefits: No Locks Needed
    - Single-threaded nature eliminates the need for locks, avoiding concurrency bugs.
6. Problem: Blocking System Calls
    - Event handlers must not make blocking calls (e.g., `open()`, `read()`), as they halt the entire server.
    - Rule: No blocking calls allowed in event-based systems.
7. Solution: Asynchronous I/O
    - APIs like `aio_read()` (on Mac) allow non-blocking I/O operations.
    - `struct aiocb` is used to specify I/O details.
    - `aio_error()` checks if an asynchronous I/O has completed.
    - Some systems use signals to notify when async I/O completes.
8. Another Problem: State Management
    - Event-based code is more complex due to "manual stack management."
    - Example: Reading from a file, then writing to a socket.
    - Solution: Use continuations (data structures to store state for future events).
9. Difficulties with Events:
    - On multi-core systems, parallel event handlers reintroduce synchronization issues.
    - Page faults can cause implicit blocking.
    - API changes (non-blocking to blocking) require code restructuring.
    - Asynchronous disk and network I/O integration can be cumbersome.
10. Summary:
    - Event-based servers offer control over scheduling but increase complexity.
    - Challenges include multi-core systems, paging, code management, and I/O integration.
    - Both threads and events persist as approaches to concurrency.
