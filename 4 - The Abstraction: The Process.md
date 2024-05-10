# The Abstraction: The process

## Key Take-aways

### Process

The process is the fundamental abstraction provided by the operating system for a running program. A process consists of the program's code, data, stack, heap, and any other state required for execution. Operating systems provide APIs like fork(), exec(), wait(), and kill() for creating, destroying, waiting for, and controlling processes.

- Fundamental abstraction provided by OS for a running program
- Consists of code, data, stack, heap, and other execution state
- OS provides APIs for creating, destroying, waiting, controlling processes
- Common APIs: fork(), exec(), wait(), kill()

### Process creation

Process creation involves loading the program code and static data from disk into memory, allocating space for the stack and heap, performing initialization, and then jumping to the main() routine. A process can be in one of several states - running (executing on the CPU), ready (ready to run but not scheduled), or blocked (waiting for I/O or an event to complete).

- Loading program code and data from disk into memory
- Allocating space for stack and heap
- Performing other initialization before jumping to main()

### Process state management

- Running - executing on CPU
- Ready - ready to run but not scheduled
- Blocked - waiting for I/O or event to complete

### Process Data structure

The OS maintains process data structures like the process list or process control blocks to track information about each process, such as the register context, state, open files, and more. Context switching is the mechanism of switching the CPU from one process to another by saving and restoring register contents and other context.

- Process list/process control blocks to track processes
- Contains register context, state, open files, etc.

By rapidly switching between processes, the OS employs time sharing to give the illusion of multiple virtual CPUs, enabling the concurrent execution of many processes. Low-level mechanisms like context switching enable process execution, while high-level scheduling policies decide which process to run when, separating the mechanisms from the policies.

Process tracing allows visualizing the execution sequence and state transitions of processes over time. With multiple processes executing concurrently, the OS must handle challenges like synchronization and race conditions introduced by concurrency issues.

- **Context Switching:**
  - Switching CPU from one process to another
  - Saving and restoring register contents and context
- **Time Sharing:**
  - Rapidly switching between processes
  - Gives illusion of multiple virtual CPUs
  - Enables concurrent execution of many processes
- **Mechanisms vs Policies:**
  - Mechanisms are low-level functionality (e.g. context switch)
  - Policies are high-level algorithms (e.g. scheduling)
- **Process Tracing:**
  - Visualizing execution sequence and state transitions over time
- **Concurrency:**
  - Multiple processes executing concurrently
  - Introduces challenges like synchronization, race conditions
  - OS must handle concurrency issues

So in essence, this chapter covers processes as the key abstraction for program execution, the process lifecycle, process state management, context switching for time sharing, and the concurrency challenges that arise from running multiple processes simultaneously.

# Answers to the Questions

1. Run process-run.py with the following flags: `-l 5:100,5:100`.What should the CPU utilization be (e.g., the percent of time the CPU is in use?) Why do you know this? Use the `-c` and `-p` flags to see if you were right.

   > **Ans:** The CPU will run until both the process finishes.Therefore CPU utilisation is 100%

2. Now run with these flags: `./process-run.py -l 4:100,1:0`. These flags specify one process with 4 instructions (all to use the CPU), and one that simply issues an I/O and waits for it to be done. How long does it take to complete both processes? Use `-c` and `-p` to find out if you were right.

   > **Ans:** The total time will be 11. (4 + IO-begin + 5(Default time to finish IO operations) + IO-end)

3. Switch the order of the processes: `-l 1:0,4:100`. What happens now? Does switching the order matter? Why? (As always, use `-c` and `-p` to see if you were right)

   > **Ans:** Yes, switching the order matters.As soon as the IO operation begins, The CPU is free for some time,Hence it utilises this time to run the other process.

4. We’ll now explore some of the other flags. One important flag is `-S`, which determines how the system reacts when a process issues an I/O. With the flag set to `SWITCH_ON_END`, the system will NOT switch to another process while one is doing I/O, instead waiting until the process is completely finished. What happens when you run the following two processes (`-l 1:0,4:100 -c -S SWITCH_ON_END`), one doing I/O and the other doing CPU work?

   > **Ans:** Since the system will NOT switch to another process while one is doing I/O, the process waits for the I/O to finish then begins executing the other process.The CPU is not used effectively!!

5. Now, run the same processes, but with the switching behavior set to switch to another process whenever one is WAITING for I/O (`-l 1:0,4:100 -c -S SWITCH-ON_IO`). What happens now? Use `-c` and `-p` to confirm that you are right.

   > **Ans:** The CPU switches to another process as soon as the previous process begins I/O. Thus using the CPU effectively!!

6. One other important behavior is what to do when an I/O completes. With `-I IO_RUN_LATER`, when an I/O completes, the process that issued it is not necessarily run right away; rather, whatever was running at the time keeps running. What happens when you run this combination of processes? `./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -c -p -I IO_RUN_LATER` Are system resources being effectively utilized?
   > **Ans:** The CPU will switch to another process as soon as the I/O starts.But, it wont return to the previous process as soon as it completes the I/O; instead finishes the current process and then comes back to the previous process.The CPU and I/O is utilised moderately.

7. Now run the same processes, but with `-I IO_RUN_IMMEDIATE` set, which immediately runs the process that issued the I/O. How does this behavior differ? Why might running a process that just completed an I/O again be a good idea?

   > **Ans:** The CPU will return to the previous process as soon as I/O finishes.The idea of running a process that just completed an I/O again is a good idea as it will completely utilise the CPU!!

8. Now run with some randomly generated processes using flags `-s 1 -l 3:50,3:50` or `-s 2 -l 3:50,3:50` or `-s 3 -l 3:50,3:50`. See if you can predict how the trace will turn out. What happens when you use the flag `-I IO_RUN_IMMEDIATE` versus the flag `-I IO_RUN_LATER` ? What happens when you use the flag `-S SWITCH_ON_IO` versus `-S SWITCH_ON_END`?
   > **Ans:** We observe that using `-I IO_RUN_IMMEDIATE` & `-S SWITCH_ON_IO` make the CPU more efficient!!
