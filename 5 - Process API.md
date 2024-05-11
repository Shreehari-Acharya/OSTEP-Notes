## Process Creation in Unix Systems

- The fork() system call is used to create a new process, which is an almost exact copy of the calling process. After fork(), there are two processes running - the parent and the child.
- The parent gets the child's PID as the return value, while the child gets 0.
- The wait() system call allows the parent to wait for the child to finish executing.
- The exec() system calls allow the child process to execute an entirely new program, breaking away from being identical to the parent.

## Motivation for fork() and exec()

- This separation of fork() and exec() enables the implementation of shells and features like I/O redirection and pipes.
- The shell forks a child, alters its environment (e.g. redirecting output), and execs the new program.

## Process Control and Users

- Signals like SIGINT (ctrl-c) and SIGTSTP (ctrl-z) can control processes.
- Each process runs as a certain "user" and users can generally only control their own processes.
- The "superuser" root can control all processes on the system.

## Tools

- ps shows running processes
- top shows processes and their resource usage
- kill sends signals to processes
- CPU monitoring tools

## Summary

- The fork()/exec()/wait() APIs provide a powerful way to create and control processes in Unix systems.

## Questions & Answers

1. Run ./fork.py -s 10 and see which actions are taken. Can you
predict what the process tree looks like at each step? Use the -c
flag to check your answers. Try some different random seeds (-s)
or add more actions (-a) to get the hang of it.

 > We have to just visualize the tree. Use the `-c` option to verify

2. One control the simulator gives you is the fork percentage, controlled by the -f flag. The higher it is, the more likely the next action is a fork; the lower it is, the more likely the action is an exit. Run the simulator with a large number of actions (e.g., -a 100) and vary the fork percentage from 0.1 to 0.9. What do you think the resulting final process trees will look like as the percent- age changes? Check your answer with -c

 > We notice hat, higher the fork percentage the more forks and less or none exit

```bash
user@linux$ ./fork.py -f 100 -c

ARG seed -1
ARG fork_percentage 100.0
ARG actions 5
ARG action_list 
ARG show_tree False
ARG just_final False
ARG leaf_only False
ARG local_reparent False
ARG print_style fancy
ARG solve True

                           Process Tree:
                               a

Action: a forks b
                               a
                               └── b
Action: a forks c
                               a
                               ├── b
                               └── c
Action: a forks d
                               a
                               ├── b
                               ├── c
                               └── d
Action: d forks e
                               a
                               ├── b
                               ├── c
                               └── d
                                   └── e
Action: d forks f
                               a
                               ├── b
                               ├── c
                               └── d
                                   ├── e
                                   └── f

```

3. Now, switch the output by using the -t flag (e.g., run ./fork.py
-t). Given a set of process trees, can you tell which actions were
taken?

```bash
user@linux$ ./fork.py -t

ARG seed -1
ARG fork_percentage 0.7
ARG actions 5
ARG action_list 
ARG show_tree True
ARG just_final False
ARG leaf_only False
ARG local_reparent False
ARG print_style fancy
ARG solve False

                           Process Tree:
                               a

Action?
                               a
                               └── b
Action?
                               a
                               ├── b
                               └── c
Action?
                               a
                               ├── b
                               ├── c
                               └── d
Action?
                               a
                               ├── b
                               ├── c
                               │   └── e
                               └── d
Action?
                               a
                               ├── b
                               ├── c
                               │   └── e
                               ├── d
                               └── f
```

>Action: a forks b
>
>Action: a forks c
>
>Action: a forks d
>
>Action: c forks e
>
>Action: a forks f

4. One interesting thing to note is what happens when a child exits; what happens to its children in the process tree? To study this, let’s create a specific example: ./fork.py -A a+b,b+c,c+d,c+e,c-. This example has process ’a’ create ’b’, which in turn creates ’c’, which then creates ’d’ and ’e’. However, then, ’c’ exits. What do
you think the process tree should like after the exit? What if you use the -R flag? Learn more about what happens to orphaned processes on your own to add more context.

```bash
user@linux$ ./fork.py -A a+b,b+c,c+d,c+e,c- -c

ARG seed -1
ARG fork_percentage 0.7
ARG actions 5
ARG action_list a+b,b+c,c+d,c+e,c-
ARG show_tree False
ARG just_final False
ARG leaf_only False
ARG local_reparent False
ARG print_style fancy
ARG solve True

                           Process Tree:
                               a

Action: a forks b
                               a
                               └── b
Action: b forks c
                               a
                               └── b
                                   └── c
Action: c forks d
                               a
                               └── b
                                   └── c
                                       └── d
Action: c forks e
                               a
                               └── b
                                   └── c
                                       ├── d
                                       └── e
Action: c EXITS
                               a
                               ├── b
                               ├── d
                               └── e

```

> We notice that when the process ‘c’ exits, the child processes of c are now the child of a. Meaning that when a process dies, if it contains any child process, it will be transferred to the root process

Now lets run the command with -R flag which will set the ARG local_reparent as True

```bash
user@linux$ ./fork.py -A a+b,b+c,c+d,c+e,c- -R -c 
ARG seed -1
ARG fork_percentage 0.7
ARG actions 5
ARG action_list a+b,b+c,c+d,c+e,c-
ARG show_tree False
ARG just_final False
ARG leaf_only False
ARG local_reparent True
ARG print_style fancy
ARG solve True

                           Process Tree:
                               a

Action: a forks b
                               a
                               └── b
Action: b forks c
                               a
                               └── b
                                   └── c
Action: c forks d
                               a
                               └── b
                                   └── c
                                       └── d
Action: c forks e
                               a
                               └── b
                                   └── c
                                       ├── d
                                       └── e
Action: c EXITS
                               a
                               └── b
                                   ├── d
                                   └── e

```

> So when we set local_reparent as True, the child processes of the exited process ‘c’ is transfered to the parent of ‘c’ (In this case process ‘b’) instead of the root process.

5. One last flag to explore is the -F flag, which skips intermediate steps and only asks to fill in the final process tree. Run ./fork.py -F and see if you can write down the final tree by looking at the series of actions generated. Use different random seeds to try this a few times.

```bash
user@linux$ ./fork.py -F

ARG seed -1
ARG fork_percentage 0.7
ARG actions 5
ARG action_list 
ARG show_tree False
ARG just_final True
ARG leaf_only False
ARG local_reparent False
ARG print_style fancy
ARG solve False

                           Process Tree:
                               a

Action: a forks b
Action: b forks c
Action: a forks d
Action: b forks e
Action: a forks f

                        Final Process Tree?
```

> use  `-c` flag to check your answer!!

```bash
Final process tree will look like:
                               a
                               ├── b
                               │   ├── c
                               │   └── e
                               ├── d
                               └── f

```

6. Finally, use both -t and -F together. This shows the final process tree, but then asks you to fill in the actions that took place. By looking at the tree, can you determine the exact actions that took place? In which cases can you tell? In which can’t you tell? Try some different random seeds to delve into this question.

```bash
user@linux$ ./fork.py -F -t

ARG seed -1
ARG fork_percentage 0.7
ARG actions 5
ARG action_list 
ARG show_tree True
ARG just_final True
ARG leaf_only False
ARG local_reparent False
ARG print_style fancy
ARG solve False

                           Process Tree:
                               a

Action?
Action?
Action?
Action?
Action?

                        Final Process Tree:
                               a
                               ├── b
                               ├── c
                               │   └── e
                               │       └── f
                               └── d

```

>Action: a forks b
>
>Action: a forks c
>
>Action: c forks e
>
>Action: e forks f
>
>Action: a forks d

>We can determine the exact actions that took place when the number of all the process is 5, if there are only a few, we cannot tell the exact order.
