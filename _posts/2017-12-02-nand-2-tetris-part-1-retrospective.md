---
layout: article
title: Nand 2 Tetris, Retrospective, Part 1
date: 2017-10-14 13:26:00
categories: programming
tags:
- programming, learning
---

I just wrapped up Chapters 1-6 of The Elements of Computing Systems and learned a ton. The book takes you from Nand gates all the way up to a fully functioning Tetris game running on software and hardware you built (the hardware is run on an emulator). By the end of Chapter 6 you've implemented some basic gates, an ALU, stateful chips like Registers and RAM, a CPU, and a pseudo "Computer" with memory-mapped keyboard and screen peripherals. As a non-credentialed programmer, reading this book and opening Pandora's box was really enlightening. If you've been programming long enough, you've heard something akin to "It's all 0's and 1'", but for me it was never really clear what that meant. I had this vague idea that something somewhere gobbled up these numbers and did something meaningful with them. Well, now I've written my own gobbler and seen it in action. Listed below are some personal highlights for me.

# Chapter 1
This chapter covers Boolean Logic and ended up being pretty fun since, as a programmer, I was already familiar with the high-level concepts (e.g. OR, AND, etc.). You're given specifications in the form of truth tables and then tasked with implementing each of the APIs in a hardware description language. I took Logic as a course in college and we had to do truth tables there so I already had some exposure, but I don't think they'd be hard to pick up for anyone. Building the gates turned out to be a little bit of a mindfuck. For example, you're expected to build the chips in the order in which they appear in the book. In this chapter that's Not, And, Or, Xor, etc. The only gate you're given to start with is a NAND gate and you somehow have to string these together to get your desired output. Well, how do you construct "OR" as a concept? Personally, I've never had to think about this since every language will give you OR for free. I won't spoil it, but it turns out you can do this using AND and NOT gates (that was my construction anyway). Besides being mind-bending, this also taught me a lot about abstraction, or at least provided a different perspective to it. Using a completely unintuitive implementation you can construct a Boolean concept that operates intuitively. Put differently, you can look at an OR gate and have a gut feel for what it does, but if I just showed you it's implementation and no output, it'd take you a while to figure out what it was doing. Good abstractions are super powerful and really let you ignore the implementation. In real-life software, I'm so used to looking at the guts of many functions that I want to use that I'd forgotten to strive for this standard.

# Chapter 2
This chapter starts off with a primer on Boolean arithmetic which was great since I knew none. I didn't even know how to translate binary numbers to decimal numbers until I saw a Dan Luu talk last year that made me think, "I should learn binary". Anyways, this chapter is mathy enough to require some focus but non-mathy enough that you can grok it without any knowledge above basic arithmetic. The concepts are just foreign (to me) and that's what makes it hard. The big chip in this portion of the book is the ALU. It takes a bunch of input and computes different functions depending on which "control bits" you toggle - imagine a function with lots of possible configurations. The whole point of this chip is so you can use it in your CPU later. Looking back, this was really cool because it maps very closely to the kinds of operations that your assembly language ends up being able to support.

# Chapter 3
To me, this is maybe the hardest chapter in the book because of the introduction of the flip flop gate. There's some background on how time needs to be discretely represented (i.e. in chunks) in computers and the explanation of what a cycle is. But I personally found this explanation confusing and had to watch the Coursera videos from the chapter to get a firmer grasp. There's some cool stuff in the videos about how the eletrical currents need time to stablize in the hardware so you can effectively ignore state in between cycles. My mental analogue here is unstaged changes in a Git repo with the clock entering a committed state by the time the next cycle begins.

Anyways, you get to build some memory chips in this chapter using the flip flop gate. As opposed to the "combinational" chips before (i.e. pure function chips), memory is all sequential chips. That stabilization thing I mentioned earlier comes into play here since after you "write" a value to a memory chip, the chip doesn't output the value until the next clock cycle. I know, it's weird. State ruins everything.

It's also here that you really start to understand what it means for a computer to be 16-bit vs 32-bit vs 64-bit. This is the number of bits that float around between the different hardware pieces and has all kinds of ramifications for the software layers built on top. FYI, the book's machine, The Hack Computer, is a 16-bit machine.

# Chapter 4
Now we move up to some software. This was my first time writing programs of even trivial length in an assembly language and it turned out to be a lot of fun. At this point, I found that I could operate at a level pretty close to my day-to-day, with a few exceptions. For example, the assembly language supports variables, branching, and even functions (sort of). The difference is that you implement these things in ways that feel a little funky but you quickly acclimate to. Oh, I need a conditional here? Set the A register to where we'll jump if our boolean check is true and figure out how to construct the comparison so that we compare against 0. After a while you grok the pattern and it's just mapping the high-level concepts to the lower-level constructs. Oh, and everything's a number. You want to access a memory address? Store the address number in the A register. You want to store the number 10? Same deal.

The other big things in this chapter were understanding exactly how your assembly code will get translated to 0's and 1's to be run by the computer and memory-mapped IO.

## Opcodes
The thing that blew my mind here was that you basically come up with a bit-pattern that corresponds to what you want your CPU to do and then you design your assembly language such that it gets translated to that. For example, let's arbitrarily say I have a computer that takes 4 bits. The 2 leftmost bits specify an arithmetic operation, like so:

```
00 | ADD
01 | SUBTRACT
10 | DIVIDE
11 | MULT
```

The 2 rightmost bits will be the operands. 0 means 0 and 1 means 1. So,

```
00 | 0 and 0
01 | 0 and 1
10 | 1 and 0
11 | 1 and 1
```

Okay, using our specifications above, to tell our imaginary computer to compute 1+1, we'd give it the "opcode" 0011. The 2 left bits say "ADD" and the 2 right ones say "1 and 1". Now, I can support 1+1 in my imaginary assembly language by having the command `ADD 1,1` translate into the opcode 0011 and then passing this pattern to the CPU. You build an assembler in chapter 6 that does exactly this but with the book's assembly language and opcodes.

## Memory mapped IO
Memory-mapped IO just means you can add peripherals to your computer by integrating them into your memory modules (this is done via drivers in real-life, I *think*). So, your keyboard is plugged in and is at let's say address 1001. To figure out what key is being pressed, inspect the memory at address 1001.

This is getting long, I'll follow-up in a different post later.
