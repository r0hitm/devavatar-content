---
lang: en
title: "Writing a mini Shell program | Shell Lab pitfalls and tips"
description: "My devlog on working through the Lab 5 - Shell Lab of 15-213: Intro to Computer Systems: Schedule for Fall 2015"
tags:
    - devlog
    - C
    - computer-architecture
pubDatetime: 2025-12-18T07:05:21.622Z
---

This is a continuation from [Going Low with x64 Assembly - Systems Programming: The Beginning](/posts/x64-assembly-csapp-intro-to-computer-systems).

## Table of contents

## Shell Lab

The Shell Lab is the Lab 5 homework of the ICS course (alternatively, also the homework problem of chapter 8 in the Computer Systems book).
The goal is to write a simple shell program that supports job control, running jobs in the foreground and background, and reacting to signals. All routines for job management are provided by the starter file. The **main crux** of the homework is spawning and managing child processes (jobs) and signal handling, particularly to write against ugly race conditions and concurrency bugs.

To be honest, I was overwhelmed by it. When I started writing the signal handlers, my test cases were always missing some things. The concurrency nature and trying to make sense of everything had me rewrite my solution from scratch three times. However, I learned something important each time. In this post, I'll write about hints and tips in case you're struggling as well.

**Note**: The code samples here are simplified for conveying the intent, so they do not do error checking. Please **ALWAYS** check for errors from syscall functions.

**Note 2**: I am assuming you have read the problem writeup thoroughly, so I will not go into explaining the homework jargon here.

## Pitfalls

### Not blocking `SIGCHLD` before forking

The lecture and the book keep repeating this, but it is still easy to forget: **Block the `SIGCHLD` before forking.**
This prevents the race condition where a child is scheduled and terminated before the parent runs, resulting in a dead job in the job list.

### Forgetting to UN-Block signals before `execve`

Forked child processes get a copy of the block vector of their parents. Therefore, you must unblock the signals you blocked during the fork in the parent, before calling `execve` to load the new program. If you don't do this, the children will keep blocking those signals. My advice would be to keep track of the previous block vector and do a `SIG_SETMASK` instead of `SIG_UNBLOCK` in `sigprocmask`.

```c
sigset_t mask, prevmask;
sigemptyset(&mask);
sigaddset(&mask, SIGCHLD);
sigprocmask(SIG_BLOCK, &mask, &prevmask);

// after fork, inside the child unblock
sigprocmask(SIG_SETMASK, &prevmask, NULL);
// call to execve()
```

### Doing data mutation in signal handlers

The rule of thumb is: **keep your signal handlers simple**. You can, however, try to mutate data in the handlers if you're sure it will not be interrupted while doing so, and the data is atomic. For this homework, you can perform `SIGTSTP` and `SIGINT` handling directly in the handlers because they just forward the signal to the child process group.

However, for `SIGCHLD`, things get complicated. The writeup hints you to do the reaping in the handler and do a busy loop wait in `waitfg`. There are multiple approaches here. After struggling a bit, I went with the `volatile sig_atomic_t sig_*` approach because it's easier to reason about mentally and keeps things simple.

I have three `sig_atomic_t` global variables initialized to 0 for each relevant signal. In the signal handler, I just set the corresponding signal to 1. In `waitfg`, I use `sigsuspend` to suspend the shell until any of the signals wake it up. Then I just check which signals have been hit and handle them appropriately synchronously.

```c
volatile sig_atomic_t g_got_sigchld = 0;
volatile sig_atomic_t g_got_sigint = 0;
volatile sig_atomic_t g_got_sigtstp = 0;

// rest of the code

// in waitfg(pid)
// ... rest of the code
    sigprocmask(SIG_BLOCK, &mask, &prevmask);

    while (pid == fgpid(jobs))
    {
        if (!g_got_sigchld && !g_got_sigtstp && !g_got_sigint)
            sigsuspend(&prevmask);

        // check for which signal woke the program and handle it appropriately
    }
    // end of waitfg()
```

**How do you handle background jobs' reaping?** Well, I have a helper that does that asynchronously with the `WNOHANG` option to `waitpid`, which I call after `eval()` returns in the main loop. This way, before showing the next prompt, any pending but terminated jobs are reaped.

### "Copying" most of the shell code from the book

If you are also following the book, there is a simpler shell implementation given by the end of chapter 8. You may be tempted to copy it and adjust it for the homework. However, that program is missing a lot of stuff you need to adjust to pass the lab.

If you're not already proficient with signal handling and process management, writing from scratch is much simpler. It helps you wrap your head around things instead of internally debating how that simple program's way of handling signals is running you into concurrency bugs.

### Not blocking signals before mutating the jobs list

On the topic of signal handling, be sure to block signals before mutating the jobs list (i.e., adding a new job, deleting a job, modifying a job). If you don't do so, there's a risk that it will be interrupted by a signal and, depending on how you're handling them, cause serious headaches. It is simple to just block all signals before doing any mutation and unblock after that.

### Using `kill(pid, sig)` instead of `kill(-pid, sig)`

I ran into this bug on test 13, and turned out I was forwarding the signal to the child process itself instead of the entire child process group.

## Debugging Tips & hints

- Run the traces in order from 1 to 16. The higher traces depend on the previous trace's success. So if trace 4 is failing, trace 5 definitely won't be correct.
- Whenever confused about how a particular user action should be handled, run the reference shell.
- Implement the verbose mode. This is an optional mode for the shell where it shows every function call and action it's performing. I'd suggest you implement it; it will help you trace back bugs, especially in signal handlers.
- **BE AWARE** of signal safety!! If you're doing more than setting some flags in your handlers, only use signal-safe functions.
- If you still feel stuck, don't forget to ask for help! [The Borr Project Community](https://borrproject.github.io/getting-help/).

<img style="margin:auto;" src="https://media1.tenor.com/m/GmNex5lZ6wUAAAAC/ghost-in.gif" />
