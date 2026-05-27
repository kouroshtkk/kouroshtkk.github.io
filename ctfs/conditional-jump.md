---
layout: default
title: "if-else in assembly"
parent: "pwn.college writeups"
date: 2026-05-27
---
# Intro

![cond-jump](/images/cond-jump.png)

In order to implement if-else in assembly you have to use multiple jumps

1. Jump to **else** if condition is not met
2. Executing if block
3. Jump to **done** to avoid executing **else**
4. Executing **else**



# Process

So we have to dereference `rdi` and compare it against `0x7f454c46`:
```nasm
mov rbx, [rdi]
cmp rdi, 0x7f454c46
```

If it is not equal we will jump to control second statement:
```nasm
jne second
#.----------------------------------
#.  first if block will go here
#.----------------------------------
jmp done
second:
cmp rbx, 0x00005A4D
jne else
#.----------------------------------
#.  second else if block will go here
#.----------------------------------
jmp done
else:
#.----------------------------------
#.  else block
#.----------------------------------
done:
```

First if block:
```nasm
mov rax, [rdi+4]
mov rcx, [rdi+8]
mov rdx, [rdi+12]
add rax, rcx
add rax, rdx
```

Second if block:
```nasm
mov rax, [rdi+4]
mov rcx, [rdi+8]
mov rdx, [rdi+12]
sub rax, rcx
sub rax, rdx
```

Else block:
```nasm
mov rax, [rdi+4]
mov rcx, [rdi+8]
mov rdx, [rdi+12]
mul rax, rcx
mul rax, rdx
```

Let's write the whole program:
```nasm
mov rbx, [rdi]
cmp rdi, 0x7f454c46
jne second
mov rax, [rdi+4]
mov rcx, [rdi+8]
mov rdx, [rdi+12]
add rax, rcx
add rax, rdx
jmp done
second:
cmp rbx, 0x00005A4D
jne else
mov rax, [rdi+4]
mov rcx, [rdi+8]
mov rdx, [rdi+12]
sub rax, rcx
sub rax, rdx
jmp done
else:
mov rax, [rdi+4]
mov rcx, [rdi+8]
mov rdx, [rdi+12]
mul rax, rcx
mul rax, rdx
done:
```

There was a error:
```console
if-else.s: Assembler messages:
if-else.s:26: Error: number of operands mismatch for `mul'
if-else.s:27: Error: number of operands mismatch for `mul'
```

We have to use **IMUL** instead of **MUL** because the data could be negative number and also it takes 2 operands.

Another error:
![qword-dword](/images/dword-qword.png)

I am loading qword from the address instead of dword so i will change the first instruction into

```nasm
mov rbx,dword ptr [rdi]
```
Type mismatch!
because the `rbx` is qword we have to load it into `ebx`.

Another error:
```console
Failed in the following way: In attempt 1, rax was expected to be 0xfffecbca, but was 0xfffffbc6fffecbca instead
```
the top part 
I think we have to zero the top part of rbx. so I know if we do `xor ebx, ebx` the top 32 bit will be set to zero(performance reasons!).

I add the following code at top:
```nasm
xor ebx,ebx
```

```console
Failed in the following way: In attempt 1, rax was expected to be 0x4ec4, but was 0x52007c725095bd56 instead 
```
I don't know what is happening, let's investigate from the beginning

Here we go in the first line we load into `ebx` and then i compare to rdi that is a non relevant pointer!
I was wrong about the xor because when I mov the dword ptr into `ebx` it automatically sets the rest of `rbx` to zero!

Still there is garbage in the upper 32 bit of the register:

```console
Failed in the following way: In attempt 1, rax was expected to be 0x80263b0c, but was 0x41ce6be980263b0c instead
```

It is because the data is dword in the rdi memory address but I am loading it into qword registers so I changed all the registers to 32 bit ones:
```nasm
.intel_syntax noprefix
.global _start
_start:
mov ebx,dword ptr [rdi]
cmp ebx, 0x7f454c46
jne second
mov eax, [rdi+4]
mov ecx, [rdi+8]
mov edx, [rdi+12]
add eax, ecx
add eax, edx
jmp done
second:
cmp ebx, 0x5a4d
jne else
mov eax, [rdi+4]
mov ecx, [rdi+8]
mov edx, [rdi+12]
sub eax, ecx
sub eax, edx
jmp done
else:
mov eax, [rdi+4]
mov ecx, [rdi+8]
mov edx, [rdi+12]
imul eax, ecx
imul eax, edx
done:
```

Finally it gave me the flag!
