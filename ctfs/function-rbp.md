---
layout: default
title: "stack pointer and rbp"
parent: "pwn.college writeups"
date: 2026-06-01
---
# Intro

![rbp-1](/images/rbp-1.png)
![rbp-2](/images/rbp-2.png)

One thing I have to understand is the indexing of **list[2]** why I have to sub 12 bytes from stack pointer:
We have 5*4 bytes space in the Stack and we have the base pointer in `rbp`.
So technically the base of array is `rbp`.
For example the `rsp` is hypothetically 400:

| Register / Offset | Address | Content |
| :--- | :--- | :--- |
| `rbp` | **400** | **Base Pointer (Start of frame)** |
| `rbp - 4` | 396 | `List[4]` |
| `rbp - 8` | 392 | `List[3]` |
| `rbp - 12` | 388 | `List[2]` |
| `rbp - 16` | 384 | `List[1]` |
| `rbp - 20 (rsp)`| **380** | `List[0]`, **Top of the Stack** |

So `[rbp-20]` is index 0,
`[rbp-16]` is index 1,
`[rbp-12` is index 2.

Formula to Calculate:
Address = Base address(rbp) + (index * element size )

For example index 4 => rbp - 20 + (4 * 4 ) = rbp - 4 ;
The array starts at the lowest memory address and goes up to arriving stack pointer.

In this program we have to look at each byte that is stored in the src_address and increment the respective index times 2 in the stack.

We multiply by two because we have maximum 0xffff of any byte; meaning we have maximum 16 bits of occurances of any individual byte.

Then we look at the saved values in the stack and and we put the byte with maximum occurance in max_freq_byte and we will return the value by putting it in `rax`.

# Process

First I will write the function body:
The two first arguments for function call are:
1. RDI: src_addr
2. RSI: size

I will use rdx as the counter for both while loops.

Size and src_addr are both used in the first while loop.

To implement the first while loop I have to compare `i` with `rsi`.

Curr_byte is only one byte so I will use `cl` first byte of `rcx`.
I put the first byte pointed by `[rdi + rdx]` in the `cl`.

To calculate the space needed for stack, I have to subtract size * 1 byte from stack pointer. 

I can't subtract registers inside bracket so I will zero extend cl and the negate it and then add to the `rbp`.

```nasm
most_common_byte:
	mov rbp, rsp
	sub rsp, rsi
	mov rdx, 0
	sub rsi, 1 # size-1
while_loop:
	cmp rdx, rsi
	ja while_done
	mov cl, byte ptr [rdi + rdx]
	movzx rcx, cl
	shl rcx, 1
	neg rcx
	inc word ptr [rbp + rcx] 
	inc rdx
	jmp while_loop
while_done:
```

The second loop:

We can use the first byte of rdx register from the first while loop to initialize **b**.

I need 2 bytes of counter for max_freq (0xffff).

I need 1 byte to store the most frequent byte.


```nasm

	mov dl, 0 # b = 0
	mov cx, 0 # max_freq = 0 ( 2 bytes )
	mov al, 0 # max_freq_byte = 0 ( 1 byte of RAX)
second_loop:
	cmp dl, 0xff
	ja second_done
	movzx r10, dl
  	shl r10, 1
	neg r10
	cmp word ptr [rbp+r10], cx
	jbe if_done
	mov cx, word ptr [rbp+r10]
	mov al, dl
if_done:
	inc dl
	jmp second_loop
second_done:
mov rsp, rbp
ret

```

The program never finishes, first problem I see is `cmp dl, 0xff`, because dl maximum value is `255` so the loop never finishes.
I change dl to `RDX`
```nasm
mov rdx, 0
.
.
.
cmp rdx,0xff
```

![rbp-error](/images/rbp-error.png)

I asked claude to review the code and there are three problems:

1. In the second loop: rdx is being modified (shifted/negated) but then used as the loop counter — it's corrupted each iteration
2. mov cx, word ptr [rbp + r10] — r10 is never set; should use the negated index to read the frequency
3. The byte value returned should be the original byte b, not the negated/shifted index

```nasm

	mov rdx, 0 # b = 0
	mov cx, 0 # max_freq = 0 ( 2 bytes )
	mov al, 0 # max_freq_byte = 0 ( 1 byte of RAX)
second_loop:
x	cmp rdx, 0xff
	ja second_done
	mov r10, rdx
  	shl r10, 1
	neg r10
	cmp word ptr [rbp+r10], cx
	jbe if_done
	mov cx, word ptr [rbp+r10]
	mov al, dl
if_done:
	inc rdx
	jmp second_loop
second_done:
mov rsp, rbp
ret

```
I ran the program 3 times and rax is always `0x0` but the counter goes up.

The problem is that I save `rsp` in `rbp` and then when index is 0 I modify the `rbp` here:
```nasm
0x400027:	inc   	word ptr [rbp + rcx]
```
and also in the second loop! and then i restore the garbage `rbp` into the `rsp` in the end! Wrong.

I changed all the `rbp` to `rsp` and the problem is solved! I also removed the `neg` parts in the code because the `rsp` is already -0x200, We could just save from -0x200 upwards. if we use `neg` we will go even below that and it is dangerous.

final code:

```nasm
---------------- CODE ----------------
0x400000:	mov   	rbp, rsp
0x400003:	sub   	rsp, 0x200
0x40000a:	mov   	rdx, 0
0x400011:	sub   	rsi, 1
0x400015:	cmp   	rdx, rsi
0x400018:	ja    	0x400030
0x40001a:	mov   	cl, byte ptr [rdi + rdx]
0x40001d:	movzx 	rcx, cl
0x400021:	shl   	rcx, 1
0x400027:	inc   	word ptr [rsp + rcx]
0x40002b:	inc   	rdx
0x40002e:	jmp   	0x400015
0x400030:	mov   	rdx, 0
0x400037:	mov   	cx, 0
0x40003b:	mov   	al, 0
0x40003d:	cmp   	rdx, 0xff
0x400044:	ja    	0x400062
0x400046:	mov   	r10, rdx
0x400049:	shl   	r10, 1
0x40004f:	cmp   	word ptr [rsp + r10], cx
0x400054:	jbe   	0x40005d
0x400056:	mov   	cx, word ptr [rsp + r10]
0x40005b:	mov   	al, dl
0x40005d:	inc   	rdx
0x400060:	jmp   	0x40003d
0x400062:	mov   	rsp, rbp
0x400065:	ret   	
0x400066:	call  	0x400000
--------------------------------------
```
