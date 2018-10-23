---
layout: article
title: Computer Architecture (CS 61C) Session 0 Notes
date: 2017-10-14 13:26:00
categories: programming
tags:
- programming, learning
---

I'm still following the curriculum outline [here](https://teachyourselfcs.com/#architecture) and since I just [finished Nand 2 Tetris](http://arlandism.github.io/programming/2017/10/14/nand-2-tetris-part-1-retrospective/), I got to start on Berkeley's Great Ideas in Computer Architecture Course. Below are my notes from the first lecture and reading from [Patterson and Hennessy](https://www.amazon.com/Computer-Organization-Design-MIPS-Fifth/dp/0124077269).

# Lecture

- Computers are diverse. They used to be room-filling monoliths, but many are now in our pockets. Also, there are huge warehouses full of computers called data centers and running them turns out to be resource-intensive because they run really hot, so they have to be near water and have these auxilary cooling systems.

- Parallelism goes all the way down the stack. You could have a computer that services parallel requests (e.g. 2 users searching at once), that delegates to parallel threads, with cpus running parallel instructions via pipelining, that are fetching pieces of data in parallel and all of this is happening in the context of hardware gates that operate in parallel.

## Catalogue of the great ideas in computer architecture
- Abstraction: Not needing to worry about how the thing below you is implemented, only its interface. I touch on how this blew my mind at the hardware level in my [last post](http://arlandism.github.io/programming/2017/10/14/nand-2-tetris-part-1-retrospective/).

- Moore's Law: This is the prediction that the number of transistors that you could fit on a chip would approximately double every 18 months. As a consequence of this, it was historically important for people architecting computing systems to factor this into their timelines. You can't design things in such a way that they don't take full advantage of new hardware capabilities by the time they're released. I wonder if there's an analogue here in software? The lecturer starts to note how Moore's law seems to be fading since the transistors are getting so small that it takes longer to get them all on the chip which in turn makes the process more expensive.

- Principle of Locality / Memory Hierarchy: This reminds me of the [Norvig line](http://norvig.com/21-days.html#answers) about remembering how long it takes to fetch data depending on where you need to go. As you move further away from the CPU, data access gets more expensive. Recognizing this and designing for it is important.

- Parallelism: See above. Software should be written to take advantage of parallelism or else you're throwing away performance. Dan Luu talks about a related idea in a really good post [here](http://danluu.com/programming-books/#computer-architecture).

- Performance Measurement and Improvement: Test your stuff to see how fast it is and try to make it better.

- Dependability via Redundancy: There's some back of the envelope math to calculate the failure rate of hard disks in a large data center. It turned about to be about one an hour. It also turns out that companies are so afraid of accidents that they'll often leave these dead disks on the racks so they don't have to worry about someone knocking out a cord when trying to remove them. But this is okay because they'll have redundant disks to take over. On a larger scale, companies also have backup data centers in the event of outages (wow, redundancy can be expensive). So, if you need to account for failures, it's a good idea to have redundancy to fall back on. Sidenote and movie spoilers: I watched the movie Passengers last night and immediately thought of a similar point. The ship basically went to shit because some of the primary systems went down and even though other nodes in the ship's computing cluster took over, they couldn't handle the work. This seems like really poor engineering for a futuristic spaceship.

## Number representation in computers

- Detour into binary and hex and how they work.
- Signed integers are represented via two's complement. To quickly convert from a positive to a negative or vice versa, invert all the bits and then add 1. It's important to be able to perform binary arithmetic to do this correctly.
- First bit in two's complement is a sign bit. 1 means negative and 0 means positive.
- If your arithmetic goes over the highest or lowest allowed integer representation in a two's complement number then it's an overflow. For example, 4 bits can represent 2^3 total numbers since you have to exclude the sign bit. You split the numbers in half between positive and negative, except you take one from the positive side to represent 0. 8 numbers, so -4...3. If you try `3+3` or `-2 - 3`, you'll overflow because you technically can't represent those numbers.
- When talking about computer addresses, you can't have negative numbers. Addresses go up from 0, so often in C you'll use unsigned ints to represent addresses. This seems like a handy idea to keep in mind when reviewing C code.

# Book

I'm not digging the book as much. In contrast to the Nand 2 Tetris book, it's written more academically with lots of fancy-sounding words and more technical definitions. On the plus side, there are lots of examples for the important stuff.

- MIPS has 32 bit words. Remember, this means the "width" of the data being passed around is 32 bits.
- You can turn say a 4 bit signed number into a 16 bit signed number by padding it with 0's to the left. In 4 bits, the number 3 is `0011`. In 16, it's `0000 0000 0000 0011`.
- There are other representation options besides two's complement, but it won out because it was easier on the hardware.

Overall, I'm pretty excited to be tackling this material and can't wait for the next lecture.
