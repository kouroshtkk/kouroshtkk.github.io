---
layout: default
title: "Jump table in assembly"
parent: "pwn.college writeups"
---
# Intro

jump table is how `switch` will get handled in assembly.
we zero all the `rax` with 

```nasm
xor eax,eax
```

:because when we use `eax` all the upper 32 bits of `rax` will be set to zeroes.

we load the char by putting the ascii value in al that is only 8 bits.
we have 256 ascii values so we have 256 indexes from the memory address we load into `rax`, indexes will be calculated by (`rax` * 8) because each entry is 8 bytes the offset is 8.

# Process

this is the code disassembled with gdb intel syntax:

```nasm
=> 0x0000000000401000 <+0>:	mov    rcx,QWORD PTR [rsp+0x10]
   0x0000000000401005 <+5>:	xor    eax,eax
   0x0000000000401007 <+7>:	mov    al,BYTE PTR [rcx]
   0x0000000000401009 <+9>:	mov    rax,QWORD PTR [rax*8+0x401085]
   0x0000000000401011 <+17>:	jmp    rax
```
with gdb command x/a256 we can list all the addresses after `0x401085`.

I found that the address 0x401013 <success> is containing the password so we will look into that address. it is +960. 960/8=120 so the index is ascii char 120 that is lowercase `"x"`

![Reading addresses](/images/1.png)

I ran the program with char x and it gave me the flag!
