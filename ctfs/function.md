---
layout: default
title: "functions in x86"
parent: "pwn.college writeups"
date: 2026-05-30
---
# Intro

![function-1](/images/function-1.png)

![function-2](/images/function-2.png)

First we have to look at function calling convention in System V AMD64:

![call-convention](/images/call-convention.png)

We have only one argument **src_addr**. So we have to put it in `RDI`.
foo function sits in **0x403000**, we have to call it inside the function.

# Process

I start by writing function labels, so I can clear my head:
```nasm
str_lower:


_start:
```
Then I assign 0 to `RDX` as we don't want to use it in function calls.
I compare the src_addr with 0, if it is 0 we jump to function done label.
I put the return value in rax following the calling convention.

```nasm
str_lower:
mov rdx, 0
cmp rdi, 0
jz func_done:
mov rax, rdx
ret
```

Now let's implement the while loop:
I have to check the src_addr byte string, if it is equal of **0x00** we jump to while done label.

Then I compare it to **0x5a**, if it is above we jump to if_done label.
I have to call foo afterwards and the first argument must be in `rdi` which is source_addr which is already in `rdi`.

Then we move the return value from the function that will be stored in `rax` based on the convention and store it in `rdi` memory address.

Now we increment i that is stored in `RDX` as written in function scope before.
inc the `rdi` that is the source addr.

After the function is finished we put the return value in `rax` that is in `rdx`.

Then ret!

```nasm
while_loop:
mov bl, byte ptr [rdi]
cmp bl, 0x00
je while_done
cmp bl, 0x5a
ja if_done:
call 0x403000
mov [rdi], rax
inc rdx
if_done:
inc rdi
jmp while_loop
while_done:
```

So let's put the while loop in the **str_lower**  function:
```nasm
str_lower:
mov rdx, 0
cmp rdi, 0
jz func_done
while_loop:
mov bl, byte ptr [rdi]
cmp bl, 0x00
je while_done
cmp bl, 0x5a
ja if_done
call 0x403000
mov byte ptr [rdi], rax
inc rdx
if_done:
inc rdi
jmp while_loop
while_done:
func_done:
mov rax, rdx
ret
```

So this is our function. We have to call the function by calling the function because `rdi` is already the first argument we need:
```nasm
_start:
call str_lower
```

![error](/images/func-error.png)

The problem is that I did not take the byte pointed in the `rdi` address to call the function!
So first I have to save `rdi` in the stack by `push rdi`, then I have to `mov dil, bl` then call the function.
After the function returns I have to `pop rdi`

The challenge crashed, It did not give me anything.

I think the problem is that I did not save `rbx` neither! so let's add the push and pop for that aswell.

![error](/images/func-error-2.png)

There is Two problems, first I don't know why although I have written call 0x403000, the program calls 0x402000. 
Second: why when I try to write to address pointed by `rdi`, `rdi` contains zero!

I got it, I have to first write the address somewhere in some register then do the call on that.
Also I have to write one byte only on the memory address so I use

`RIP` calculates the call address by `effective address = rip + displacement` that is why we got 0x402000

```nasm
mov byte ptr [rdi], al
```

Another crash:
```console
Your program has crashed!
Invalid memory write (UC_ERR_WRITE_UNMAPPED)
This was the state of your program on crash:
+--------------------------------------------------------------------------------+
| Registers                                                                      |
+-------+----------------------+-------+----------------------+------------------+
|  rax  |  0x0000000000404361  |  rbx  |  0x0000000000000041  |                  |
|  rcx  |  0x0000000000000000  |  rdx  |  0x0000000000404318  |                  |
|  rsi  |  0x0000000000000000  |  rdi  |  0x0000000000000000  |                  |
|  rbp  |  0x0000000000000000  |  rsp  |  0x00007fffff1ffff8  |                  |
|  r8   |  0x0000000000000000  |  r9   |  0x0000000000000000  |                  |
|  r10  |  0x0000000000000000  |  r11  |  0x0000000000000000  |                  |
|  r12  |  0x0000000000000000  |  r13  |  0x0000000000000000  |                  |
|  r14  |  0x0000000000000000  |  r15  |  0x0000000000000000  |                  |
|  rip  |  0x0000000000400029  |       |                      |                  |
+---------------------------------+-------------------------+--------------------+
| Stack location                  | Data (bytes)            | Data (LE int)      |
+---------------------------------+-------------------------+--------------------+
| 0x00007fffff1ffff8 (rsp+0x0000) | 09 31 40 00 00 00 00 00 | 0x0000000000403109 |
+---------------------------------+-------------------------+--------------------+
| Memory location                 | Data (bytes)            | Data (LE int)      |
+---------------------------------+-------------------------+--------------------+
|    0x0000000000404318 (+0x0000) | 41 4b 6c 41 56 43 6f 58 | 0x586f4356416c4b41 |
|    0x0000000000404320 (+0x0008) | 75 64 67 57 79 75 41 65 | 0x6541757957676475 |
|    0x0000000000404328 (+0x0010) | 74 77 59 5a 55 56 4b 52 | 0x524b56555a597774 |
|    0x0000000000404330 (+0x0018) | 75 77 74 59 5a 77 49 55 | 0x5549775a59747775 |
|    0x0000000000404338 (+0x0020) | 54 48 6f 70 51 68 65 55 | 0x55656851706f4854 |
|    0x0000000000404340 (+0x0028) | 47 4d 53 44 43 77 65 67 | 0x6765774344534d47 |
|    0x0000000000404348 (+0x0030) | 52 77 52 56 48 51 78 63 | 0x6378514856527752 |
|    0x0000000000404350 (+0x0038) | 45 57 4c 57 67 6a 6d 77 | 0x776d6a67574c5745 |
|    0x0000000000404358 (+0x0040) | 70 52 52 6a 6b 49 7a 7a | 0x7a7a496b6a525270 |
|    0x0000000000404360 (+0x0048) | 48 46 4f 4a 41 4e 49 55 | 0x55494e414a4f4648 |
+---------------------------------+-------------------------+--------------------+
```
Looking at the program state `rdx` has strange data in it, and it is because I have done pop in the reverse order of push. THE STACK IS LIFO!!!

```nasm
push rdi         ; Pushed FIRST
push rdx         ; Pushed SECOND
mov dil, bl
mov rax, 0x403000
call rax
pop rdi          ; BUG! I have to pop rdx first!
pop rdx          ; BUG! 
```

Finally my code worked! this is the final code:

```nasm
.intel_syntax noprefix
.global _start
str_lower:
	mov rdx, 0
	cmp rdi, 0
	jz func_done
while_loop:
	mov bl, byte ptr [rdi]
	cmp bl, 0x00
	je while_done
	cmp bl, 0x5a
	ja if_done
	push rdi
	push rdx
	mov dil, bl
	mov rax, 0x403000
	call rax
	pop rdx
	pop rdi
	mov byte ptr [rdi], al
	inc rdx
	if_done:
	inc rdi
	jmp while_loop
while_done:                                                                                                                                                                                
func_done:
	mov rax, rdx
	ret
	
_start:
	call str_lower
```

Lesson learned:
1. Stack is LIFO, we have to pop in reverse order we pushed!
2. Size matters in loading and storing in memory addresses.
3. We have to store the registers in the calling convention in the stack which we are using, when calling a function.

