---
layout: default
title: "integer division in x86"
parent: "pwn.college writeups"
date: 2026-05-25
---
# Intro

![the problem](/images/integer-div.png)
So as far as i understood the division in x86 happens like this:
we have **128 bit** dividend and **64 bit** divisor at maximum.
div does something intersting saving the remainder and the quotient at the same time.
you should put upper 64 bits of dividend in `rdx` and lower 64 bit in `rax`:
`rax = rdx:rax / reg`
so result is in `rax` and `rdx` is the remainder.

	
# Process

So if we want to calculate `speed = distance / time` we should put `distance` in `rax` because it is smaller than 64 bits and set `rdx` to zero!
i like this technique so i set `rdx` to zero like this:

```nasm
xor edx, edx
```

and then we move the `rdi` into the `rax`

```nasm
mov rax,rdi
```

and we would do: 

```nasm
div rsi
```
I think this will solve the problem so i would try and it gave me the flag!
