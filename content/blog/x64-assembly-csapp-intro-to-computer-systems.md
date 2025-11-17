---
title: "Going Low with x64 Assembly - Systems Programming: The Beginning"
description: "I started learning about systems programming with CS:APP book and ICS course, after completing Algorithms specialization"
tags:
    - C
    - assembly-language
    - computer-architecture
    - review
pubDatetime: 2025-09-06T18:16:15.000Z
modDatetime: 2025-10-03T07:45:28.759Z
---

[Last time](/posts/course-review-coursera-tim-roughgarden-algorithms-npcomplete), I was thinking hard about my next steps after completing
the Algorithms Specialization. And, after procrastinating by solving some LeetCode problems, I decided to start learning systems programming.

My main reasons are: I already know C, my college background is in electronics, I have experience programming in 8085 assembly, and I want to develop PC/mobile apps. Also, I plan to take OSTEP after this - I will write my own OS - so excited. A fundamental knowledge of how computers work will definitely come in handy there.

## Table of contents

## 15-213 Intro to Computer Systems + CS:APP

I'm doing the [15-213 Intro to Computer Systems](https://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/index.html), and [CS:APP](https://csapp.cs.cmu.edu/). So far, it's going great!

The course/book starts with an overview of computer architecture, then dives into how data/numbers are represented in computers - the unsigned shenanigans in C, and how it's all modular arithmetic.

Then it starts diving deep into machine languages, x86-64 assembly (and C). My academic background is in electronics, and I have programmed 8085 chips in assembly. It didn't take much time for me to pick up on x86-64 assembly. Though, they are using AT&T assembly syntax, and I am used to Intel syntax from 8085. Their order of operands is reversed, and there are some minor changes in how they represent registers and dereference addresses. At the machine level, it's all the same thing, no changes; it's just how they get represented in assembly which throws me off sometimes.

Also, new things that I've learned so far include the `objdump` command that can disassemble an executable, extract symbols, and getting more familiar with using GDB. At the time of writing this, I'm doing homework 2 - bomblab - reverse-engineering an executable to find six secret phrases that will defuse the bomb. It was super overwhelming when I began last week, but today I have found 3 codes so far. And it has become super fun! Analyzing the machine/assembly instructions, following the flow of logic, trying to reason about things while keeping an eye on the processor state and flags. This properly taught me how the processor handles argument passing beyond the registers (spoilers: it uses the stack) between procedures, and calling conventions properly, which I, unfortunately, skipped in my college classes.

### Update 2025-09-15

I solved bomblab! Phew, it was hard. The moment you have to decode usage of a data structure, it gets complicated real fast, in my opinion.

### Update 2025-10-03

I completed the `attacklab`. Phew! It was challenging but fun. This lab involved using buffer-overflow attacks to make the machine execute our (potentially malicious) arbitrary code.

The first three phases were "simple" in that the vulnerable program was compiled with modern security features disabled, such as stack address randomization and a non-executable stack. However, it still required creativity to construct an exploit string, especially with a custom value to modify registers and run our code.

Phase 4 enabled these security features and required the use of Return-Oriented Programming (ROP) attacks. With ROP, we use the program's own hex sequences (which correspond to desired instructions) to construct a series of addresses. This sequence is then used in a buffer overflow to jump to these arbitrary addresses on the stack, execute an instruction, return, and repeat. Finding these "gadgets" is the most difficult and tedious part. Once you have a list of gadgets that can be arranged to execute your desired logic, constructing the exploit string is relatively easy.

Overall, I had fun with playing as a novice hacker ahahaha 🧑‍💻.

~~I will keep this blog post updated as I make progress. So keep an eye out for that!~~

> Every software is open-source if you can read assembly.
