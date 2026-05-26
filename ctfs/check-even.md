---
layout: default
title: "check if even"
parent: "pwn.college writeups"
date: 2026-05-26
---
# Intro

The challenge is to set `y` to 1 if the `rdi` is even and 0 if it is odd.

![check-even](/images/check-even.png)

so in order to control if a number is even i can think of doing an `or` with immediate value of `0`.
if the result is 1 it should jump to the else part and if it is zero we set `rax` to 1.

# Process

I searched and or sets the zero flag in x86 so we can jump if the zero flag is not set:

```nasm
	or rdi, 0
	jnz else
	mov rax, 1
else:
	mov rax, 0
```

Classic error: 

![jne-not-allowed](/images/jne-not-allowed.png)

I think i can do an xor with `rax`, it just came to my mind but i will write to see what happens:
for example we have a odd number 0b0101, i set `rax = 1` then i would `xor` that with the `or` result of `rdi` with `0`.

if it was odd like our example the `xor` result would be `0` so rax is set to zero, otherwise it will be set to `1`

```text
or 0b0101 0 => result = 1
mov rax, 1
xor rax, rdi
```

Another classic error:

```console
ERROR: fail: this instruction is not allowed: mov
```

I can do `or rax, 1`

Error!

![wrong](/images/something-really-wrong.png)

I am doing something really wrong!

I know i can set `rax` to `0` completely by doing `xor eax, eax` because this will set all upper 32 bits to zero aswell.
Then i will or with 1 so i have `rax` set to 1.

If i do an `and rdi, 1` i will have the LSB in the `rdi` so i can perform the final `xor`

```nasm
xor eax, eax
and rdi, 1
or rax, 1
xor rax,rdi
```

finally it worked!
