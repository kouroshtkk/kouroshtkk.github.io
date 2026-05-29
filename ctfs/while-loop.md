---
layout: default
title: "while loop average in x86"
parent: "pwn.college writeups"
date: 2026-05-29
---
# Intro

![count-non-zero](/images/count-non-zero.png)

In this challenge I have to calculate the average of contiguos non zero datas saved in the memory location saved in the `rdi`.
Basically the thing I have to do is really similar to the for-loop challenge but with a diffrence in comparison, we don't have counter we have a condition that needs to be met.

The pointer is `rdi` and we have to increment the offset by **1** because we are reading byte sized data. So i will use **BYTE PTR**.
I need to store it in a byte sized register so I will use byte part of `rbx`, `bl`.

# Process

I have to first check if `RDI` is 0 or not. If it is zero we set `rax` equal to 0 and we are done!

```nasm
test rdi, rdi
jez set_rax_done
set_rax_done:
mov rax, 0
```

Let's get to the while loop logic:
I have to load the byte stored in `rdi` then compare it to 0, if it is not zero we increment the `rax` (we have to zero rax with `xor rax, rax`). 
If it is zero we jump to the **done** label without touching `rax`.
I increment rdi by 1 and we do the loop again.


```nasm
xor rax,rax
test rdi, rdi
jz set_rax_done
while_loop:
mov bl, BYTE PTR [rdi]
cmp bl, 0
jz done
inc rax
inc rdi
jmp while_loop
set_rax_done:
mov rax, 0
done:
ret
```

![error-while-loop](/images/error-while-loop.png)

The problem is because of that redundant ret at the end, it is not necessary on this challenge, because it pops off a memory address from stack pointer and jumps to it.
But in this challenge stack is not set up.
