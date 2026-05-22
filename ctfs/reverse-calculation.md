---
layout: default
title: "Reverse the Calculation"
parent: "pwn.college writeups"
date: 2026-05-22
---
# Intro

In this challenge the program does some math to the character you entered.
I will disassemble the program using `objdump -d -M intel`:
```nasm
Disassembly of section .text:

0000000000401000 <_start>:
  401000:	48 8b 44 24 10       	mov    rax,QWORD PTR [rsp+0x10]
  401005:	80 00 61             	add    BYTE PTR [rax],0x61
  401008:	80 38 95             	cmp    BYTE PTR [rax],0x95
  40100b:	75 62                	jne    40106f <fail>
  40100d:	c6 04 24 2f          	mov    BYTE PTR [rsp],0x2f
  401011:	c6 44 24 01 66       	mov    BYTE PTR [rsp+0x1],0x66
  401016:	c6 44 24 02 6c       	mov    BYTE PTR [rsp+0x2],0x6c
  40101b:	c6 44 24 03 61       	mov    BYTE PTR [rsp+0x3],0x61
  401020:	c6 44 24 04 67       	mov    BYTE PTR [rsp+0x4],0x67
  401025:	c6 44 24 05 00       	mov    BYTE PTR [rsp+0x5],0x0
  40102a:	48 89 e7             	mov    rdi,rsp
  40102d:	48 c7 c6 00 00 00 00 	mov    rsi,0x0
  401034:	48 c7 c0 02 00 00 00 	mov    rax,0x2
  40103b:	0f 05                	syscall
  40103d:	48 89 c7             	mov    rdi,rax
  401040:	48 89 e6             	mov    rsi,rsp
  401043:	48 c7 c2 40 00 00 00 	mov    rdx,0x40
  40104a:	48 c7 c0 00 00 00 00 	mov    rax,0x0
  401051:	0f 05                	syscall
  401053:	48 89 c2             	mov    rdx,rax
  401056:	48 c7 c7 01 00 00 00 	mov    rdi,0x1
  40105d:	48 c7 c0 01 00 00 00 	mov    rax,0x1
  401064:	0f 05                	syscall
  401066:	48 c7 c0 3c 00 00 00 	mov    rax,0x3c
  40106d:	0f 05                	syscall

000000000040106f <fail>:
  40106f:	48 c7 c0 3c 00 00 00 	mov    rax,0x3c
  401076:	0f 05                	syscall
```

# Process

so it looks like the `argv[1]` is loaded into `rax` and program adds `0x61` to the `ASCII` value and then compares that against `0x95` so the password must be the character `0x95-0x61` that would be `0x34` that is `ASCII` char of number `4`, so i would test that with the program:
It successfuly gave me the flag.

