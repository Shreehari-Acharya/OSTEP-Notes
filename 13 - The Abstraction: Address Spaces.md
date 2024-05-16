# The Abstraction: Address Spaces

## 1. Early Systems

In the early days of computing, systems did not provide much abstraction to users regarding memory. The physical memory looked like this:

```
max
64KB ----------------
                      Current Program
                      (code, data, etc.)
0KB  ----------------
     |                Operating System
     |                (code, data, etc.)

```

- There was just one user program running in physical memory along with the OS routines.
- Users didn't expect much in terms of ease of use, performance, or reliability.

## 2. Multi programming and Time Sharing

As machines became more expensive, **multi-programming** was introduced to improve CPU utilization. Multiple processes were ready to run, and the OS would switch between them when one process performed I/O.

The era of **time sharing** followed, allowing multiple users to interactively use the machine concurrently. Problems:

- Simply swapping entire processes in and out of memory was too slow, especially as memory sizes grew.
- Need for **protection** - processes should not interfere with each other's memory.

## 3. The Address Space

The **address space** is the running program's view of memory. It contains:

- **Code segment**: Instructions of the program
- **Heap segment**: Dynamically allocated memory (e.g., via `malloc()`)
- **Stack segment**: Local variables, function call info, etc.

Example:

```
16KB ----------------
         |            Stack (grows upward)
         |
         |
2KB      ------------
         |
1KB      ------------ Heap (grows downward)
         |
0KB      ------------ Program Code

```

The OS creates an **illusion** of a large, private address space for each process by mapping virtual addresses used by the process to physical addresses in memory.

## 4. Goals of Virtual Memory

1. **Transparency**: Virtual memory should be invisible to the program.
2. **Efficiency**: Virtualization should have low time and space overhead.
3. **Protection**: Processes should be isolated from each other and the OS.

## 5. Linux Memory Tools

- `free`: Shows total, used, and free memory in the system.
- `pmap`: Shows the memory map of a process, including code, data, shared libraries, etc.

The chapter explains how operating systems evolved from basic memory abstraction to virtual memory, which provides the illusion of a large, private address space to each process. Virtual memory achieves transparency, efficiency, and protection through hardware and software mechanisms. The homework involves using Linux tools like `free` and `pmap` to explore virtual memory usage.

## Questions & Answers

1. **The first Linux tool you should check out is the very simple tool free. First, type `man free` and read its entire manual page; it’s short, don’t worry!**
    
    **Ans:** Reading the man pages
    

1. **Now, run free, perhaps using some of the arguments that might be useful (e.g., -m, to display memory totals in megabytes). How much memory is in your system? How much is free? Do these
numbers match your intuition?**
    
    ```bash
                total       used      free      shared     buff/cache   available
    Mem:        15963       6190      2835      674        6937         8814
    Swap:       8063        0         8063
    ```
    

1. **Next, create a little program that uses a certain amount of memory, called `memory-user.c`. This program should take one command line argument: the number of megabytes of memory it will use. When run, it should allocate an array, and constantly stream through the array, touching each entry. The program should do this indefinitely, or, perhaps, for a certain amount of time also specified at the command line.**
    
    ```c
    #include <stdio.h>
    #include <stdlib.h>
    
    int main(int argc, char* argv[]) {
        if (argc != 2) {
            fprintf(stderr, "Usage: %s <memory_in_mb>\n", argv[0]);
            return 1;
        }
    
        long long mem_bytes = atoll(argv[1]) * 1024 * 1024; 
        char* memory = malloc(mem_bytes);
    
        while (1) {
            for (int i = 0; i < mem_bytes; i += 4096) {
                memory[i] = 0; // Keep accessing the memory
            }
        }
    
        free(memory);
        return 0;
    }
    ```
    

1. **Now, while running your memory-user program, also (in a different terminal window, but on the same machine) run the free tool. How do the memory usage totals change when your program is running? How about when you kill the memory-user program? Do the numbers match your expectations? Try this for different amounts of memory usage. What happens when you use really large amounts of memory?**
    
    **Ans:** Running `free -m` while `memory-user` is running with different memory usage amounts will show the "used" memory increasing accordingly. After killing `memory-user`, the used memory should decrease.
    
2. **Let’s try one more tool, known as pmap. Spend some time, and read the pmap manual page in detail.**
    
    **Ans:** read the man page. `man pmap` 
    

1. **To use pmap, you have to know the process ID of the process you’re interested in. Thus, first run `ps auxw` to see a list of all processes; then, pick an interesting one, such as a browser. You can also use your memory-user program in this case (indeed, you can even have that program call getpid() and print out its PID for your convenience).**
    
    **Ans:** note down the pid of any interesting process.
    

1. **Now run pmap on some of these processes, using various flags (like -X) to reveal many details about the process. What do you see? How many different entities make up a modern address space, as opposed to our simple conception of code/stack/heap?**
    
    **Ans:** We can see the address space is made up of many mapped regions, including:
    
    - The executable code itself (Firefox)
    - Shared libraries like `libpthread.so`, `ld.so` etc.
    - Anonymous private memory mappings (`rw---`) for the heap, stack, etc.
    - Mapped files/data sections
    - And more...
    
    The address space on modern systems is highly segmented and consists of many different mapped regions with varying permissions, rather than just a simple code/stack/heap layout.
    
2. **Finally, let’s run pmap on your memory-user program, with different amounts of used memory. What do you see here? Does the output from pmap match your expectations?**
    
    Run memory-user programming with 800
    
    ```bash
    User@Linux:~$ pmap 4630
    4630:   ./mem_usage 800
    00005dc0cd90d000      4K r---- mem_usage
    00005dc0cd90e000      4K r-x-- mem_usage
    00005dc0cd90f000      4K r---- mem_usage
    00005dc0cd910000      4K r---- mem_usage
    00005dc0cd911000      4K rw--- mem_usage
    00005dc0cf27c000    132K rw---   [ anon ]
    00007bd8aba00000 819204K rw---   [ anon ] <---------- SPACE ASSIGNED HERE!!
    00007bd8ddc00000    160K r---- libc.so.6
    00007bd8ddc28000   1568K r-x-- libc.so.6
    00007bd8dddb0000    316K r---- libc.so.6
    00007bd8dddff000     16K r---- libc.so.6
    00007bd8dde03000      8K rw--- libc.so.6
    00007bd8dde05000     52K rw---   [ anon ]
    00007bd8ddf61000     12K rw---   [ anon ]
    00007bd8ddf76000      8K rw---   [ anon ]
    00007bd8ddf78000      4K r---- ld-linux-x86-64.so.2
    00007bd8ddf79000    172K r-x-- ld-linux-x86-64.so.2
    00007bd8ddfa4000     40K r---- ld-linux-x86-64.so.2
    00007bd8ddfae000      8K r---- ld-linux-x86-64.so.2
    00007bd8ddfb0000      8K rw--- ld-linux-x86-64.so.2
    00007ffe0d50f000    132K rw---   [ stack ]
    00007ffe0d5b5000     16K r----   [ anon ]
    00007ffe0d5b9000      8K r-x--   [ anon ]
    ffffffffff600000      4K --x--   [ anon ]
     total           821888K
    
    ```
    
    800MB is 800000KB
    
    Run memory-user program with 500
    
    ```bash
    User@Linux:~$ pmap 4753
    4753:   ./mem_usage 500
    000056ce29f00000      4K r---- mem_usage
    000056ce29f01000      4K r-x-- mem_usage
    000056ce29f02000      4K r---- mem_usage
    000056ce29f03000      4K r---- mem_usage
    000056ce29f04000      4K rw--- mem_usage
    000056ce2a5bf000    132K rw---   [ anon ]
    00007cac51e00000 512004K rw---   [ anon ] <---------- SPACE ASSIGNED HERE!!
    00007cac71400000    160K r---- libc.so.6
    00007cac71428000   1568K r-x-- libc.so.6
    00007cac715b0000    316K r---- libc.so.6
    00007cac715ff000     16K r---- libc.so.6
    00007cac71603000      8K rw--- libc.so.6
    00007cac71605000     52K rw---   [ anon ]
    00007cac71796000     12K rw---   [ anon ]
    00007cac717ab000      8K rw---   [ anon ]
    00007cac717ad000      4K r---- ld-linux-x86-64.so.2
    00007cac717ae000    172K r-x-- ld-linux-x86-64.so.2
    00007cac717d9000     40K r---- ld-linux-x86-64.so.2
    00007cac717e3000      8K r---- ld-linux-x86-64.so.2
    00007cac717e5000      8K rw--- ld-linux-x86-64.so.2
    00007ffd5e46a000    132K rw---   [ stack ]
    00007ffd5e560000     16K r----   [ anon ]
    00007ffd5e564000      8K r-x--   [ anon ]
    ffffffffff600000      4K --x--   [ anon ]
     total           514688K
    ```
    
    500MB is 500000KB
