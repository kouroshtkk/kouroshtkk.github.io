---
layout: default
title: "functions in x86"
parent: "pwn.college writeups"
date: 2026-06-01
---
# Intro

![rbp-1](/images/rbp-1.png)
![rbp-2](/images/rbp-2.png)

One thing I have to understand is the indexing of **list[2]** why I have to sub 12 bytes from stack pointer:
We have 5*4 bytes space in the Stack and we have the base pointer in `rbp`.
So technically the base of array is `rbp`.
For example the `rsp` is hypothetically 400:
| rsp | 400 | rbp |
| rsp - 4 | 396 | List[4] |
| rsp - 8 | 392 | List[3] |
| rsp - 12 | 388 | List[2] |
| rsp - 16 | 384 | List[1] |
| rsp - 20 | 380 | List[0], Top of the Stack | 

So `[rbp-20]` is index 0,
`[rbp-16]` is index 1,
`[rbp-12` is index 2.

Formula to Calculate:
Address = Base address(rbp) + (index * element size )

For example index 4 => rbp - 20 + (4 * 4 ) = rbp - 4 ;
The array starts at the lowest memory address and goes up to arriving stack pointer.
To be continued...

# Process

