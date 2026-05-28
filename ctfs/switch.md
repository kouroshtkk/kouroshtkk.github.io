---
layout: default
title: "Implementing switch statements in x86"
parent: "pwn.college writeups"
date: 2026-05-28
---
# Intro

[Other challenge with switch statement](/jump-table.md/)

I have to implement this:
![switch](/images/switch.png)

Jump table address will be stored in `rsi` and number is in `rdi`. 

# Process

First I need to calculate the offset, so I will multiply the number in `rdi` by 8 and then add that to the base address in `rsi` and then jump.

```nasm
imul rdi, 8
add rsi, rdi
jmp rsi
```
![switch-error](/images/switch-error.png)

I forgot to dereference the `rsi`

![switch-error](/images/switch-error-2.png)

Again error: I forgot that if we get a number more than 3 we should always jump to the maximum default memory address. 

I will load address in `rax` and then add the default value to it.

`ja` and `jg` needs a relative address only `jmp` can jump to an absolute address.
So i will use a `jbe` (jump below equal) to the rest of the switch statement and if not jump to the default absolute address.


```nasm
mov rax, rsi
add rax, 0x18
cmp rdi, 2
jbe .skip_default
jmp [rax]
.skip_default:
imul rdi, 8
add rsi, rdi
jmp [rsi]
```

![switch-error](/images/switch-error-3.png)

The expected result is 1 less than the needed result in `rdi`.

The problem is that we have **4** entries not 3, so i have to jump to `0x20` not `0x18` and compare with `3` not with `2`.

fixed code:

```nasm
.intel_syntax noprefix
.global _start
_start:
        mov rax, rsi
        add rax, 0x20
        cmp rdi, 3
        jbe .skip_default
        jmp [rax]
.skip_default:
        imul rax, rdi, 8
        add rsi, rax
        jmp [rsi]
```
