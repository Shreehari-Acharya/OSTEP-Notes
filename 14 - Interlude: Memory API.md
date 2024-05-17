# Interlude: Memory API

## Types of Memory

- **Stack Memory**
    - Allocated/de-allocated automatically by compiler
    - Limited lifetime, de-allocated when function returns
    - Used for local variables
- **Heap Memory**
    - Explicitly allocated/freed by programmer using `malloc()`/`free()`
    - Persists beyond function lifetime
    - Requires careful memory management

## malloc() Call

- `void *malloc(size_t size);`
- Allocates requested `size` bytes on heap
- Returns pointer to allocated memory or `NULL` if failed
- Use `sizeof()` operator to request proper size for data types
    - `int *x = malloc(sizeof(int));`
- Be careful with `sizeof` a pointer vs entire allocated block

## free() Call

- `void free(void *ptr);`
- De-allocates memory previously allocated by `malloc()`
- Pass pointer returned by `malloc()`
- Failing to `free()` causes memory leaks

## Common Errors

- **Forgetting to Allocate**
    - `char *str; strcpy(str, "hello");` // Segfault
- **Not Allocating Enough**
    - `char *str = malloc(strlen("hello"));` // No space for null terminator
- **Not Initializing Memory**
    - Using uninitialized values can cause weird behavior
- **Forgetting to free()**
    - Memory leaks - exhausts available memory
- **Freeing Before Done**
    - Dangling pointer to freed memory
- **Double free**
    - Freeing same memory twice - undefined behavior
- **Invalid free()**
    - Passing pointer not from `malloc()`

## Other Calls

- `calloc()` - Allocates and zeroes memory
- `realloc()` - Resizes previously allocated memory

## Underlying Support

- `malloc()`/`free()` are library calls, not system calls
- Use system calls like `brk()`, `sbrk()`, `mmap()` to request more memory from OS

## Tools

- **gdb** - Debugger to inspect program state
- **valgrind** - Detects memory errors like leaks, invalid accesses

## Best Practices

- Use tools like `gdb`, `valgrind` to catch memory bugs
- Learn proper memory management to build robust programs
- Develop good habits - allocate, use, then free all memory

## Questions & Answers:

1. **First, write a simple program called null.c that creates a pointer to an integer, sets it to NULL, and then tries to de-reference it. Compile this into an executable called null. What happens when you run this program?**

```c
#include<stdlib.h>
#include<stdio.h>

int main(){
int *x = NULL;
printf("value of x:%p",*x);
return 0;
}
```

```bash
User@Linux:~$ ./null 
Segmentation fault (core dumped)
```

Answer: When you run this program, it will crash due to a segmentation fault. This is because the program is trying to de-reference a NULL pointer, which is not a valid location in memory.

2.  **Next, compile this program with symbol information included (with the -g flag). Doing so let’s put more information into the executable, enabling the debugger to access more useful information about variable names and the like. Run the program under the debugger by typing gdb null and then, once gdb is running, typing run. What does gdb show you?**

```bash
Program received signal SIGSEGV, Segmentation fault.
0x0000555555555161 in main () at null.c:6
6	printf("value of x:%p",*x);
```

Answer: When running the program under gdb, it shows you the line where the program crashed. In this case, it crashed at line 6, which is the line where the program is trying to de-reference a NULL pointer. This confirms our initial explanation for the segmentation fault.

3. **Finally, use the valgrind tool on this program. We’ll use memcheck that is a part of valgrind to analyze what happens. Run this by typing in the following: `valgrind --leak-check=yes ./null` What happens when you run this? Can you interpret the output from the tool?**

```bash
User@Linux:~$ valgrind --leak-check=yes ./null 
==7245== Memcheck, a memory error detector
==7245== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==7245== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==7245== Command: ./null
==7245== 
==7245== Invalid read of size 4
==7245==    at 0x109161: main (null.c:6)
==7245==  Address 0x0 is not stack'd, malloc'd or (recently) free'd
==7245== 
==7245== 
==7245== Process terminating with default action of signal 11 (SIGSEGV)
==7245==  Access not within mapped region at address 0x0
==7245==    at 0x109161: main (null.c:6)
==7245==  If you believe this happened as a result of a stack
==7245==  overflow in your program's main thread (unlikely but
==7245==  possible), you can try to increase the size of the
==7245==  main thread stack using the --main-stacksize= flag.
==7245==  The main thread stack size used in this run was 8388608.
==7245== 
==7245== HEAP SUMMARY:
==7245==     in use at exit: 0 bytes in 0 blocks
==7245==   total heap usage: 0 allocs, 0 frees, 0 bytes allocated
==7245== 
==7245== All heap blocks were freed -- no leaks are possible
==7245== 
==7245== For lists of detected and suppressed errors, rerun with: -s
==7245== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
Segmentation fault (core dumped)
```

4. **Write a simple program that allocates memory using malloc() but forgets to free it before exiting. What happens when this program runs? Can you use gdb to find any problems with it? How about valgrind (again with the --leak-check=yes flag)?**

Program code:

```c
#include<stdlib.h>
#include<stdio.h>

int main(){
int *x = (int *) malloc(10 * sizeof(int)); // allocating memory for 10 int
printf("Memory allocated\n");
return 0;
}
```

Program runs without any errors:

```bash
user@Linux:~$ ./memalloc 
Memory allocated
```

Using gdb:

```bash
user@Linux:~$ gdb ./memalloc 
GNU gdb (Ubuntu 15.0.50.20240403-0ubuntu1) 15.0.50.20240403-git
Copyright (C) 2024 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./memalloc...
(gdb) run
Starting program: /home/shreehari/memalloc 

This GDB supports auto-downloading debuginfo from the following URLs:
  <https://debuginfod.ubuntu.com>
Enable debuginfod for this session? (y or [n]) y
Debuginfod has been enabled.
To make this setting permanent, add 'set debuginfod enabled on' to .gdbinit.
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Memory allocated
[Inferior 1 (process 8271) exited normally]
(gdb) 
```

gdb does not give us any warning or sign that we have not freed the memory

Using valgrind:

```bash
user@Linux:~$ valgrind --leak-check=yes ./memalloc 
==8327== Memcheck, a memory error detector
==8327== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==8327== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==8327== Command: ./memalloc
==8327== 
Memory allocated
==8327== 
==8327== HEAP SUMMARY:
==8327==     in use at exit: 40 bytes in 1 blocks
==8327==   total heap usage: 2 allocs, 1 frees, 1,064 bytes allocated
==8327== 
==8327== 40 bytes in 1 blocks are definitely lost in loss record 1 of 1
==8327==    at 0x4846828: malloc (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==8327==    by 0x10917E: main (memalloc.c:5)
==8327== 
==8327== LEAK SUMMARY:
==8327==    definitely lost: 40 bytes in 1 blocks
==8327==    indirectly lost: 0 bytes in 0 blocks
==8327==      possibly lost: 0 bytes in 0 blocks
==8327==    still reachable: 0 bytes in 0 blocks
==8327==         suppressed: 0 bytes in 0 blocks
==8327== 
==8327== For lists of detected and suppressed errors, rerun with: -s
==8327== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)

```

 valgrind does inform that  we have not freed the memory and also the size.

5. **Write a program that creates an array of integers called data of size 100 using malloc; then, set data[100] to zero. What happens when you run this program? What happens when you run this program using valgrind? Is the program correct?**

Program code:

```c
#include<stdio.h>
#include<stdlib.h>

int main(){
int *arr = (int *) malloc(100 * sizeof(int));
arr[100] = 0;
return 0;
}
```

Running the Program:

```bash
user@Linux:~$ ./data 
Assigned value 0 to arr[100]
```

The program successfully executed, without  any errors

Using valgrind:

```bash
User@Linux:~$ valgrind --leak-check=yes ./data 
==8648== Memcheck, a memory error detector
==8648== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==8648== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==8648== Command: ./data
==8648== 
==8648== Invalid write of size 4
==8648==    at 0x10918D: main (data.c:6)
==8648==  Address 0x4a811d0 is 0 bytes after a block of size 400 alloc'd
==8648==    at 0x4846828: malloc (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==8648==    by 0x10917E: main (data.c:5)
==8648== 
Assigned value 0 to arr[100]
==8648== 
==8648== HEAP SUMMARY:
==8648==     in use at exit: 400 bytes in 1 blocks
==8648==   total heap usage: 2 allocs, 1 frees, 1,424 bytes allocated
==8648== 
==8648== 400 bytes in 1 blocks are definitely lost in loss record 1 of 1
==8648==    at 0x4846828: malloc (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==8648==    by 0x10917E: main (data.c:5)
==8648== 
==8648== LEAK SUMMARY:
==8648==    definitely lost: 400 bytes in 1 blocks
==8648==    indirectly lost: 0 bytes in 0 blocks
==8648==      possibly lost: 0 bytes in 0 blocks
==8648==    still reachable: 0 bytes in 0 blocks
==8648==         suppressed: 0 bytes in 0 blocks
==8648== 
==8648== For lists of detected and suppressed errors, rerun with: -s
==8648== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)

```

==8648== Invalid write of size 4
==8648==    at 0x10918D: main (data.c:6)

This error tells us that we are trying to access out of bound address

6. **Create a program that allocates an array of integers (as above), frees them, and then tries to print the value of one of the elements of the array. Does the program run? What happens when you use valgrind on it?**

Program Code:

```c
#include<stdio.h>
#include<stdlib.h>

int main(){
int *arr = (int *) malloc(100 * sizeof(int));
printf("Allocated memory for an array of size 100\n");
free(arr);
printf("freed the memory\n");
printf("The value at arr[50] is:%d\n", arr[50]);
return 0;
}
```

Running the program:

```bash
User@Linux:~$ ./prog
Allocated memory for an array of size 100
freed the memory
The value at arr[50] is:0
```

Using valgrind

```bash
User@Linux:~$ valgrind --leak-check=yes ./prog
==9954== Memcheck, a memory error detector
==9954== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==9954== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==9954== Command: ./prog
==9954== 
Allocated memory for an array of size 100
freed the memory
==9954== Invalid read of size 4
==9954==    at 0x1091F7: main (prog4.c:9)
==9954==  Address 0x4a81108 is 200 bytes inside a block of size 400 free'd
==9954==    at 0x484988F: free (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==9954==    by 0x1091DD: main (prog4.c:7)
==9954==  Block was alloc'd at
==9954==    at 0x4846828: malloc (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==9954==    by 0x1091BE: main (prog4.c:5)
==9954== 
The value at arr[50] is:0
==9954== 
==9954== HEAP SUMMARY:
==9954==     in use at exit: 0 bytes in 0 blocks
==9954==   total heap usage: 2 allocs, 2 frees, 1,424 bytes allocated
==9954== 
==9954== All heap blocks were freed -- no leaks are possible
==9954== 
==9954== For lists of detected and suppressed errors, rerun with: -s
==9954== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
```

==9954== Invalid read of size 4
==9954==    at 0x1091F7: main (prog4.c:9)
==9954==  Address 0x4a81108 is 200 bytes inside a block of size 400 free'd

Once again valgrind has showed the error

7. **Now pass a funny value to free (e.g., a pointer in the middle of the array you allocated above). What happens? Do you need tools to find this type of problem?**

Program Code:

```c
#include<stdio.h>
#include<stdlib.h>

int main(){
int *arr = (int *) malloc(100 * sizeof(int));
printf("Allocated memory for an array of size 100\n");
printf("Trying to free by providing a funny value like : free(arr+50)\n");
free(arr+50);
return 0;
}
```

Running Program:

```bash
User@Linux:~$ ./funny 
Allocated memory for an array of size 100
Trying to free by providing a funny value like : free(arr+50)
free(): invalid pointer
Aborted (core dumped)
```

We get an error. No we do not need tool to find this type of problems.

8. **Try out some of the other interfaces to memory allocation. For example, create a simple vector-like data structure and related routines that use realloc() to manage the vector. Use an array to store the vectors elements; when a user adds an entry to the vector, use realloc() to allocate more space for it. How well does such a vector perform? How does it compare to a linked list? Use valgrind to help you find bugs**

```c
#include <stdio.h>
#include <stdlib.h>

int* vector = NULL;
int vector_size = 0;
int vector_capacity = 0;

void vector_push(int value) {
    if (vector_size == vector_capacity) {
        // Reallocate with double the capacity
        vector_capacity = vector_capacity ? vector_capacity * 2 : 2;
        vector = realloc(vector, vector_capacity * sizeof(int));
    }
    vector[vector_size++] = value;
}

int main() {
    vector_push(1);
    vector_push(2);
    vector_push(3);
    vector_push(4);
    vector_push(5);

    printf("Vector: ");
    for (int i = 0; i < vector_size; i++) {
        printf("%d ", vector[i]);
    }
    printf("\n");

    free(vector);
    return 0;
}
```
