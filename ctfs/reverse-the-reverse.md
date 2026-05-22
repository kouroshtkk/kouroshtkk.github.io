---
layout: default
title: "Reverse the Reverse"
parent: "pwn.college writeups"
date: 2026-05-22
---
# Intro

In this challenge the program does some math to the character you entered like the last challenge i did. but this time it will subtract from the char you entered.
I will disassemble the program using `objdump -d -M intel`:
```nasm
Disassembly of section .text:

0000000000401000 <_start>:
  401000:	48 8b 44 24 10       	mov    rax,QWORD PTR [rsp+0x10]
  401005:	80 28 20             	sub    BYTE PTR [rax],0x20
  401008:	80 38 4e             	cmp    BYTE PTR [rax],0x4e
  40100b:	75 62                	jne    40106f <fail>
  40100d:	c6 04 24 2f          	mov    BYTE PTR [rsp],0x2f
  401011:	c6 44 24 01 66       	mov    BYTE PTR [rsp+0x1],0x66
  401016:	c6 44 24 02 6c       	mov    BYTE PTR [rsp+0x2],0x6c
  40101b:	c6 44 24 03 61       	mov    BYTE PTR [rsp+0x3],0x61
  401020:	c6 44 24 04 67       	mov    BYTE PTR [rsp+0x4],0x67
  401025:	c6 44 24 05 00       	mov    BYTE PTR [rsp+0x5],0x0
```

# Process

so it looks like the `argv[1]` is loaded into `rax` and program subs `0x20` from the `ASCII` value and then compares that against `0x4e` so the password must be the character `0x4e+0x20` that would be `0x6e` that is `ASCII` char of lowercase `n`, so i would test that with the program:
It successfuly gave me the flag.

