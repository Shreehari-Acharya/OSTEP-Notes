## Communication Basics

### Unreliable Networks

- Communication in distributed systems is fundamentally unreliable
- Packets can be lost, corrupted, or not reach their destination
- Causes include bit flips, damaged hardware, and buffer overflow in routers/switches

### Dealing with Unreliability

- Some applications handle packet loss themselves
- UDP is an example of an unreliable messaging layer
- TCP provides reliable communication on top of unreliable networks

## Reliable Communication Layers

### Acknowledgments (Acks)

- Receiver sends back a short ack message to confirm receipt
- Allows sender to know message was received

### Timeouts and Retries

- Sender sets a timer when sending a message
- If no ack received before timeout, assumes message was lost
- Sender retries by sending the message again

### Sequence Numbers

- Used to detect duplicate messages
- Sender and receiver maintain a counter
- Allows receiver to know if it has already seen a message

## Remote Procedure Call (RPC)

### Key Components

- Stub generator - automates packing of function arguments into messages
- Runtime library - handles communication details

### Stub Generator

- Takes interface of exported server functions
- Generates client and server stub code
- Client stub marshals arguments, sends message, waits for reply, unmarshals results
- Server stub unmarshals arguments, calls function, marshals results, sends reply

### Runtime Library Challenges

- Service location/naming
- Choice of transport protocol (TCP vs UDP)
- Handling long-running calls
- Byte ordering differences
- Fragmentation of large messages
- Asynchronous calls

## Other Considerations

- Checksums for data integrity
- Careful timeout value selection
- Server concurrency models (e.g. thread pools)
- End-to-end argument for reliability

The key takeaway is that distributed systems must be designed to handle failures as a common occurrence, with communication protocols and abstractions like RPC helping to mask the unreliability of the underlying network.
