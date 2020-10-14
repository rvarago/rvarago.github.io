---
layout: "post"
title:  "Handling Zombie Processes in Linux"
tags:   linux
---

> What a zombie process in the Linux system is, how it arises and how to kill them for good.

* * *

A Linux process is an instance of a program that is executing in the OS with all the required resources (e.g. CPU and memory). Every process has an associated number called Process Identifier (PID) that uniquely identifies a process in Linux's processes table.

We say that a process becomes a zombie when it has completed its task, but the parent process hasn't collected its execution status. The process is already sort of "dead", most of its resources have been released. It's just "waiting for the reaper".

The critical issue related to a zombie is its occupation of a PID, so
preventing the OS from assigning this PID to other processes. Few zombies might not be that harmful, however, as the number quickly grows, it may surpass the maximum allowed number of PIDs, and thus we incur the risk of being out of available PIDs. Therefore, preventing the OS from starting other processes.

# How to Kill a Zombie Process?

Actually: **you can't**, because the zombie is already dead! It's just
waiting for reaper (give back its PID). The problem lies in its parent when it hasn't collected its child's execution status. Hence, the issue is: How can we enforce that parents collect their child execution status?

There are two options:

1\. Send a signal of type SIGCHLD to the parent process asking it to do so.  
2\. Kill the parent process.

The option 1 is a request and it may or may not be honoured, depending on the
implementation of the program.

Whereas option 2 is more severe, but also more assured to work. But how
exactly it works? When you kill the parent process, its child becomes an
orphan process that is adopted by the _init_ process (PID = 1).

The crucial point is: _init_ periodically collects the execution status from
its children, hence the zombie can finally  "rest in peace" and give away
their PID to the processes table.

## Example

The following example was compiled on Linux 4.4.0-104-generic, with GCC 5.4.0, and easily distribute the example, I used a [Docker
container]({{ site.baseurl }}{% link _posts/2018-05-10-a-quick-introduction-to-docker.md %}).

The example consists of a program that will create a child process that will exit, whereas the parent process runs in an infinite loop and doesn't collect its child's execution status and hence let it becomes a zombie.

Thus, we can test option (2), which is killing the parent process,
waiting for the `init` process to adopt the orphan child process, and then collecting its status.

Typically, a process creates its child by calling the syscall `fork` that duplicates its memory content into two separated memory spaces and,
besides other things, the child will have its own PID and a Parent PID (PPID) that will be set the PID of its parent process.

The following C code does what we need (**disclaimer**: it's just a toy program that hasn't any real utility, other than illustrating our goal:

<script src="https://gist.github.com/rvarago/ee3beec3fbc6b338aab725469d715c4b.js"></script>

The `fork` syscall creates a child process and both (parent and child)
processes will execute the line following _fork_. To distinguish between parent and child, we can check the value returned from `fork`:

  * `< 0`: failure
  * `== 0`: child process
  * `> 0`: parent process (returns the child's PID)

First, let's inspect the absence of zombie processes by issuing the _top_
command:

![](/assets/img/2018-07-13-handling-zombie-processes-in-linux_0.png)

Now, compile and run the above program:
   
    
    gcc -o zombie-test create-zombie.c && ./zombie-test &

The output should look similar to:

![](/assets/img/2018-07-13-handling-zombie-processes-in-linux_1.png)

And it can be analyzed with the `ps` utility:

    
    
    ps aux | grep test-zombie

The output should be similar:

![](/assets/img/2018-07-13-handling-zombie-processes-in-linux_2.png)

That shows that two processes match the searching
pattern. Notice that the child process (PID = 14) has a _Z_ status, and it's marked as _defunct_, which means that the child has become a zombie.

You can double-check the fact that this process has become a zombie by running `top` again:

![](/assets/img/2018-07-13-handling-zombie-processes-in-linux_3.png)

Looking at the zombie's _stat_ you can see that wee have one zombie process.

Try to kill the child process by (SIGTERM is the default signal and hence it can be omitted):
    
    
    kill <CPID>

Where `<CPID>` is the child's PID (14 in my case).

We should notice that nothing has happened (check by running `ps` once more).

To send a SIGCHLD to the parent process:
    
    
    kill -SIGCHLD <PPID>

Where `<PPID>` is the parent's PID (13 in my case).

Notice that the zombie is still alive. It happens because we're not treating the SIGCHLD signal.

Therefore, the way to reap the child is to kill its parent by:
    
    kill <PPID>

Now, by killing the parent process, as we've said, the `init` process will
adopt the orphaned child process, periodically collect its execution status, and then reap the child; which gives back its PID to the Linux process table.

Running `top` again and we should see that the zombie has been reaped _(zombie == 0)_:

![](/assets/img/2018-07-13-handling-zombie-processes-in-linux_4.png)

Looking at the code, specifically at the comment (1), the proper way to avoid a zombie is to uncomment the call to the `wait` syscall at the process so that it can collect the child's execution status.

The parameter of `wait` is a pointer to an integer that will be
filled with the child's return value, so that the parent can inspect it. In our example, we're ignoring it by passing `NULL`.

By this explanation, we know more closely what _init_ is doing when adopting the zombie orphans: it's continuously calling _wait_ on its children.

# Conclusion

I hope that you now have a better understanding of what a zombie process on Linux is, and how to get away from them.

Although it was not the main point, we saw a brief example of multiprocessing programming on Linux using the C programming language and the `fork` syscall.

I encourage you to search for more information about concurrent and parallel programs, its pros and cons, and how to use the C API to develop on Linux.

# References

[1] <https://kerneltalks.com/howto/everything-need-know-zombie-process/>

[2] <http://man7.org/linux/man-pages/man2/fork.2.html>

[3] <http://man7.org/linux/man-pages/man2/waitpid.2.html>

[4] <http://man7.org/linux/man-pages/man2/syscalls.2.html>

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
