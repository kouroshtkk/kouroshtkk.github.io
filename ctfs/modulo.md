---
layout: default
title: "modulo of divisor 2^n in x86"
parent: "pwn.college writeups"
date: 2026-05-25
---
# Intro

In this challenge i have to calculate modulo of `rax = rdi % 256` and `rbx = rsi % 65536`

there is an interesting thing in binary that if you want to calculate a modulo of some number by another number that is **2^n** for example 
```text
186 % 128
```
`128` is `2^7` **n** is `7`, you only have to look at the lower `n` bits of first operand.

```text
186 = 0b10111010
```
so the first 7 bits are `0xb111010` that is `58`, computers always surprise me!

# Process

So 256 is `2^8`, we have to look at lower 8 bits of `rdi` and put it in `rax`
lower 8 bits of `rdi` is `dil` and the lower 16 bits (2^16=65536) of `rsi` is `si`.
as you can see here:


![registers](/images/registers.png)


```nasm
mov rax, dil
mov rbx, si
```
okay i got the error:

```console
eff-mod.s:4: Error: operand type mismatch for `mov'
eff-mod.s:5: Error: operand type mismatch for `mov'
```
and i have forgotten that the data in registers with diffrent size can not be moved around.

I have to write `movzx` to zero extend the rest of the register.

```nasm
movzx rax, dil
movzx rbx, si
```

I got another error:

![not-allowed](/images/not-allowed.png)

I thought if i move to the destination registers with the same size that would work!

```nasm
mov al, dil
mov bx, si
```
and it worked! 
I learned a thing or two in this excersize that i tought is not worth writing anything about.
