---
layout: default
title: "xor reverse"
parent: "pwn.college writeups"
date: 2026-05-22
---
# Intro
In this challenge the program gets the `argv[1]` char and the `xor`s with the specific value:
this is what part of the assembly looks like:

```nasm
0000000000401000 <_start>:
  401000:	48 8b 44 24 10       	mov    rax,QWORD PTR [rsp+0x10]
  401005:	80 30 67             	xor    BYTE PTR [rax],0x67
  401008:	80 38 3d             	cmp    BYTE PTR [rax],0x3d
  40100b:	75 62                	jne    40106f <fail>
```

# Process

so the program loads the char into `rax` and then does `xor` with `0x67` and then compares with `0x3d` so as we know if we `xor` `0x3d` with `0x67` we get the value requested by the program:

```text
0x3d: 00111101
0x67: 01100111
xor : 01011010 that is: 0x5a
```
let's test `0x5a` that is `ASCII` value of uppercase letter `Z`:
good news! that is correct.

