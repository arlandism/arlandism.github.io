---
layout: article
title: Pangram Detector in x86
date: 2021-03-30 10:52:43
categories: programming
tags:
- programming
- x86
---

I'm currently learning x86 assembly as part of my Bradfield coursework. I just wrapped up a fun exercise, which was to write a program to detect whether a string used every single letter of the alphabet. The most common example is "The quick brown fox jumps over the lazy dog".

## Devising a scheme

In the land of assembly, we have registers and memory. And at this level of abstraction it makes more sense to think in terms of bit patterns. To start, what's the best way to represent a letter? Well, letters are just numbers encoded with meaning. So, looking at an ASCII or UTF-8 table, you can see that the decimal value for 'A' is 65. There are 26 letters in the alphabet, ranging from decimal values 65-90 in uppercase. I have 64 bits at my disposal so if I just subtract 65 from every letter I can encode the entire alphabet in the lower 26 bits.

But what about lowercase letters? They have a separate decimal value, ranging from 97-122. Well, ASCII was designed so that the offset of a lowercase letter is exactly it's uppercase equivalent + 32. This means I can just subtract 32 from a lowercase value and it'll be equivalent.

Finally, handling punctuation and spaces ends up being easy as well. After performing both of the normalization steps above (subtracting 32 and 65), I can just check the number I end up with. If it's greater than 0x3ffffff (which is all of the lower 26 bits flipped), then I can just ignore it.

## Implementing said scheme

The code is [here](https://github.com/arlandism/x86-fun/blob/master/pangram.asm). I tried to avoid branching instructions where possible and only ended up with 2, so I was happy about that.

I ran into a couple of annoying issues. For example, shift instructions only work with immediate values or things in the `cl` register. And I kept screwing up the hex conversions - the lower 26 bits all flipped is 0x3ffffff, *not* the number 26 encoded in hex: 0x1a. I also hit a `bus error` with a memory fetch that went too far (my first assembly array bounds error!).

I figured out some other useful things too. Since `mov` instructions don't modify the `rflags` register, I can safely queue up intermediate values in registers and then chain together `cmp` instructions:

```x86
  mov r8, 0x1
  cmp rax, 0x3ffffff
  cmove rax, r8
  mov r8, 0x0
  cmovnz rax,r8
```

If rax == 0x3ffffff then return true (1), otherwise return false (0). This makes control flow a lot easier.

## Conclusions

This is the most advanced assembly I've ever written so I'm pretty excited. That being said, I do recognize it's somewhat trivial once the encoding scheme is figured out. I'm also still not super comfortable with all of the nuances (e.g. what `shl` is allowed to use, performance best practices, nasm best practices, etc.). But overall this was super fun and there's a certain rush you get when you get a working program at this level of abstraction.
