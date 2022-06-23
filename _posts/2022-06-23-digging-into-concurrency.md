---
layout: article
title: digging-into-concurrency
date: 2022-06-23 15:12:29
categories: programming
tags:
- programming
---

I'm in between jobs at the moment, so decided it was a good time to improve my knowledge base. My new job will be oriented around cost. When I worked at Remind we started a big initiative to reduce cost and looked a lot at CPU utilization to understand where and by how much we could resize our fleet. In reflecting on this, I remember having lots of questions about what CPU utilization _actually meant_, so I thought maybe it was a good time to revisit [The Meteor Book](https://pages.cs.wisc.edu/~remzi/OSTEP/) and somehow I winded up in the Concurrency section...

## The Motivation

Why might you want multiple threads to begin with? One big motivation right now is that we're increasingly moving towards a multi-processor world. In a CPU there's a register that points to the address of the next instruction, which is then fed into the CPU pipeline to be processed. If you have another set of instructions that you want to run those will have to wait until the OS scheduler decides it's time to context switch (usually via a CPU interrupt - this is called pre-emptive scheduling). But...if you have another CPU that's idle, then the situation is different. That other set of instructions could be running there at the same time the original set of instructions is running on the first CPU. This is a form of parallelism and is desirable because 1) you can theoretically get more work done in the same amount of time, and 2) you're utilizing more of your system's resources (that second CPU isn't sitting idle).

Now to be clear, this benefit of parallelism is something you can also get with multiple processes. So why threads in particular? One notable difference is that threads share the same address space as the process from which they were spawned. Each thread has its own stack (which btw needs to be saved/restored on context switches). But because of this shared address space idea, threads can share data structures with one another. Imagine an http process that has an in-memory cache but wants to be able to process multiple requests at once.

TODO: I suspect there are other tradeoffs I'm not thinking of and I'd love to check out some real systems to understand their design decisions. antirez's decision not to make Redis multi-threaded comes to mind.

## The Challenge

So we want to be able to leverage multiple CPUs and share data structures. What's the problem? The tricky part is modifying data structures across threads can result in unexpected behavior. For example, check out this C code:

```C
#include<stdio.h>

void myfunc(int *x) {
  *x = *x + 1; // get value of x from memory, increment it, and write it back
}

int main() {
  int a = 1; // allocate a variable on the stack
  myfunc(&a); // pass in the var's address
  printf("a is %d\n", a);
}

```

It turns out there's a lot happening when we add 1 to x, even though, from the programmer's perspective, it's just a single line.  You can see that in the following disassembly (obtained via objdump -d):

```asm
0000000100003f30 _myfunc:
100003f3c: 8b 08                       	movl	(%rax), %ecx
100003f3e: 83 c1 01                    	addl	$1, %ecx
100003f45: 89 08                       	movl	%ecx, (%rax)
```

Some lines are omitted but the sequence shown here is:
1. Move a value from memory (addr in `%rax`) into the `%ecx` register
2. Increment what's in the `%ecx` register and write it back
3. Move the value in `%ecx` back to memory

The trouble with this in the context of multiple threads is that the operating system can decide to stop one thread in the middle of its execution and schedule another and we have no control over this. So the 3 steps above could be interleaved with another thread running the same sequence of steps and the final ordering is indeterminate. For example, imagine 2 threads T1 and T2. This could be a possible ordering:

```
tick
   1 T1 movl	(%rax), %ecx
   (interrupt)
   2 T2 movl	(%rax), %ecx
   3 T2 addl	$1, %ecx
   4 T2 movl	%ecx, (%rax)
   (interrupt)
   5 T1 addl	$1, %ecx
   6 T1 movl	%ecx, (%rax)
```
(Remember that each thread has its own stack and gets its registers reset on a context switch)

In this example, the final value of `a` would be 2 even though we'd want and expect it to be 3.

## The Solution

What we need here is a way to say, "wait your turn" when it comes to updating the shared variable. This is where locks come in. Hopefully I'll get to those in another post.

## TODO:
- locks
- linux futexes
- linux vs solaris lock implementations
- atomic hardware instructions
