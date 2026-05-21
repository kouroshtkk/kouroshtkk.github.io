---
layout: default
title: "Loop in assembly"
parent: "pwn.college writeups"
date: 2026-05-21
---
# Intro
This challenge compares the `argv[1]` with the password to give you the flag.
this is what disassembly looks like:
it looks like the program loads the `argv[1]` that is pointed by `rsp+16` into `rdi`, then stores a string in rsp
```nasm
0000000000401000 <_start>:
  401000:	48 8b 7c 24 10       	mov    rdi,QWORD PTR [rsp+0x10]
  401005:	c6 04 24 73          	mov    BYTE PTR [rsp],0x73
  401009:	c6 44 24 01 6f       	mov    BYTE PTR [rsp+0x1],0x6f
  40100e:	c6 44 24 02 48       	mov    BYTE PTR [rsp+0x2],0x48
  401013:	c6 44 24 03 79       	mov    BYTE PTR [rsp+0x3],0x79
  401018:	c6 44 24 04 32       	mov    BYTE PTR [rsp+0x4],0x32
  40101d:	c6 44 24 05 39       	mov    BYTE PTR [rsp+0x5],0x39
  401022:	c6 44 24 06 00       	mov    BYTE PTR [rsp+0x6],0x0
  401027:	48 8d 34 24          	lea    rsi,[rsp]

000000000040102b <loop>:
  40102b:	8a 06                	mov    al,BYTE PTR [rsi]
  40102d:	3a 07                	cmp    al,BYTE PTR [rdi]
  40102f:	75 6e                	jne    40109f <fail>
  401031:	3c 00                	cmp    al,0x0
  401033:	74 08                	je     40103d <success>
  401035:	48 ff c7             	inc    rdi
  401038:	48 ff c6             	inc    rsi
  40103b:	eb ee                	jmp    40102b <loop>

000000000040103d <success>:
  40103d:	c6 04 24 2f          	mov    BYTE PTR [rsp],0x2f
  401041:	c6 44 24 01 66       	mov    BYTE PTR [rsp+0x1],0x66
  401046:	c6 44 24 02 6c       	mov    BYTE PTR [rsp+0x2],0x6c
  40104b:	c6 44 24 03 61       	mov    BYTE PTR [rsp+0x3],0x61
  401050:	c6 44 24 04 67       	mov    BYTE PTR [rsp+0x4],0x67
  401055:	c6 44 24 05 00       	mov    BYTE PTR [rsp+0x5],0x0
  40105a:	48 89 e7             	mov    rdi,rsp
  40105d:	48 c7 c6 00 00 00 00 	mov    rsi,0x0
  401064:	48 c7 c0 02 00 00 00 	mov    rax,0x2
  40106b:	0f 05                	syscall
  40106d:	48 89 c7             	mov    rdi,rax
  401070:	48 89 e6             	mov    rsi,rsp
  401073:	48 c7 c2 40 00 00 00 	mov    rdx,0x40
  40107a:	48 c7 c0 00 00 00 00 	mov    rax,0x0
  401081:	0f 05                	syscall
  401083:	48 89 c2             	mov    rdx,rax
  401086:	48 c7 c7 01 00 00 00 	mov    rdi,0x1
  40108d:	48 c7 c0 01 00 00 00 	mov    rax,0x1
  401094:	0f 05                	syscall
  401096:	48 c7 c0 3c 00 00 00 	mov    rax,0x3c
  40109d:	0f 05                	syscall

000000000040109f <fail>:
  40109f:	48 c7 c0 3c 00 00 00 	mov    rax,0x3c
  4010a6:	0f 05                	syscall
```

# Process

