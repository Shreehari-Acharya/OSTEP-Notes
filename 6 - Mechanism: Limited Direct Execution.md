### Limited Direct Execution
- OS runs user programs directly on CPU in user mode with restrictions
- System calls allow requesting privileged operations from kernel in kernel mode
- OS sets up trap table with handlers for different events like system calls, interrupts
### Restricted Operations
- Programs trap into kernel via system calls for privileged operations like I/O
- Kernel validates arguments and performs operation if allowed
- Prevents processes from bypassing security/access controls
### Switching Between Processes
- Timer interrupts allow OS to regain control periodically
- Cooperative approach relies on processes voluntarily giving up CPU (system calls, yield)
- Non-cooperative approach uses timer interrupts to preempt processes
### Context Switching
- OS decides to switch from one process to another
- Saves current process registers/state in process control block (PCB)
- Restores register values from new process PCB
- Changes kernel stack pointer to new process stack
- Timer interrupt causes implicit hardware register save to kernel stack
- Context switch code saves/restores kernel registers explicitly
### Hardware Support
- CPU has at least 2 modes - user (restricted) and kernel (privileged)
- Trap instruction transfers control to kernel, saving registers
- Return-from-trap instruction goes back to user mode after system call
- OS sets up trap table addresses during boot
### Concurrency Considerations
- OS needs to handle interrupts during interrupt processing
- May disable interrupts temporarily during critical sections
- Use locking schemes to protect shared kernel data structures
### Summary
- Limited direct execution allows efficient execution with OS control
- Timer interrupt and context switching enable timesharing
- Hardware support for CPU modes, system calls is crucial
    - OS must handle concurrency issues carefully
