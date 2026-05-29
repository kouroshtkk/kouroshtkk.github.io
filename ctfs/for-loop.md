---
layout: default
title: "for loop average in x86"
parent: "pwn.college writeups"
date: 2026-05-29
---
# Intro

![for-loop](/images/for-loop.png)
The memory address of first quad word is stored in `rdi`.
Since quad word is 64 bits, the offset is 8 bytes.

# Process

I will put the **i** value in `rbx`. 
The sum value in `rcx`.
```nasm
xor rax,rax
mov rbx, 1 # i=1
mov rcx, 0 # sum = 0
loop:
cmp rbx,rsi # if i(rbx) is above n (rsi)
ja .done
add rax, qword [rdi] 
add rdi, 8 # set the offset to the next qword 
add rbx, 1 # i+=1
jmp loop
done:
xor rdx,rdx # to clear top 64 bit of rdx:rax ( we only need rax )
div rsi # rdx:rax / rsi, result goes to rax
```

![for-loop-error](/images/for-loop-error.png)

There is a strange thing happening:
```nasm
0x400016:	add   	rax, qword ptr [rdi + 8]
```
It is reading `rdi+8` in the first run, but in my code I have written `add rax, qword [rdi]`

I changed it to `add rax, qword ptr [rdi]` and it worked finally, but i still don't know why.
So let's search:
I found a reply by **Peter Cordes** on StackOverflow:
[SackOverFlow reply](https://stackoverflow.com/questions/2987876/what-does-dword-ptr-mean#comment137844672_2987916)

![peter](/images/Peter.png)

