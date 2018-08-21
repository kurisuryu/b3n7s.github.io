---
layout: post
title:  "Strange function prologues"
date:   2018-08-21 22:33:00 +0200
categories: 
---

I noticed that on a recent Debian 9 machine, wich gcc version 6.3.0, a new prologue and epilogue. Let's try to find its utility.

We will use a simple test program:

```
#include <stdio.h>

int sub(int x){
	return x*2;
}

int main(int argc, char *argv[]){
	printf("You have %d arguments\n", argc);
	sub(5);
	return 0;
}
```

and compile it with:

```
$ gcc -m32 -o test test.c
```

Checking the disassembly can be done with:

```
$ objdump -d -M intel test
```

We get the following prologue in `main`:
```
 846:	8d 4c 24 04          	lea    ecx,[esp+0x4]
 84a:	83 e4 f0             	and    esp,0xfffffff0
 84d:	ff 71 fc             	push   DWORD PTR [ecx-0x4]
 850:	55                   	push   ebp
 851:	89 e5                	mov    ebp,esp
```

and following epilogue in `main`:

```
 5fc:	5d                   	pop    ebp
 5fd:	8d 61 fc             	lea    esp,[ecx-0x4]
 600:	c3                   	ret 
```

Why? Also, our `sub` function has a classic prologue and epilogue:

```
 5a0:	55                   	push   ebp
 5a1:	89 e5                	mov    ebp,esp
...... 
 5b2:	5d                   	pop    ebp
 5b3:	c3                   	ret  
```

GCC seems to only insert this new prologue and epilogue in the `main` function. Again, why?

Turns out the answer is: to align the stack at the beginning of `main` on a 16-byte boundary. This is mandatory if you have SSE instructions acting on stack variables. This is not the case in our simple program, so it is not mandatory.

We can change this behavior with a compiler flag `-mpreferred-stack-boundary` (expressed as a power of 2):

```
gcc -m32 -mpreferred-stack-boundary=2 -o test test.c
```

When we look now at the prologue and epilogue of `main`, we indeed see the classic versions:

```
 5b4:	55                   	push   ebp
 5b5:	89 e5                	mov    ebp,esp
.... 
 5e8:	c9                   	leave  
 5e9:	c3                   	ret    
```

## Prologue details

Let us now check the details of the "weird" prologue and epilogue:

```
 5b4:	8d 4c 24 04          	lea    ecx,[esp+0x4]
```

`ecx` is loaded with the address of the first argument to `main` (i.e. `argc`).

```
 5b8:	83 e4 f0             	and    esp,0xfffffff0
```

`esp` is lowered to its nearest 16-byte boundary address.

```
 5bb:	ff 71 fc             	push   DWORD PTR [ecx-0x4]
```

`ecx-0x4` equals `esp+0x4-0x4` and thus `esp` before the stack correction and points thus to the return address of `main`.

Basically we are pushing the return address (again) on the stack.

```
 5be:	55                   	push   ebp
 5bf:	89 e5                	mov    ebp,esp
```

From here on our classic prologue starts. So all that this has done, is change our stack pointer to a 16-byte boundary and recreated a classic stack frame there, i.e. saved frame pointer + return address.

## Between prologue and epilogue

If the stack pointer was changed, how to know where my arguments of my main function are? Well, we have a pointer to our first argument in `ecx`! So `ecx` will be used to find the arguments to `main`.

## Epilogue details

An epilogue has to do 3 things:

* Restore `esp`
* Restore `ebp`
* Return

This is done in 3 instructions:

```
 5fc:	5d                   	pop    ebp
```

Restores `ebp`. It was indeed pushed to the stack at the end of our prologue (technically second last instruction of prologue).

```
 5fd:	8d 61 fc             	lea    esp,[ecx-0x4]
```

Remember that `ecx` is still pointing at the first argument to `main`. So `ecx-0x4` points at the return address of `main`.
We now set `esp` to point to this return value, just before executing our:

```
 600:	c3                   	ret  
```

## Why only in `main`?

Finally, we can ask ourselved why we only align the stack in `main` on 16-byte boundaries. The reason is that for other functions, the caller is responsable for assuring the alignment.

Take for example when `main` calls `sub`: it is the responsability of `main` to make sure the stack frame of `sub` is well aligned. It does this by modifying `esp` before pushing the argument(s) for the function call:

```
 5e5:	83 ec 0c             	sub    esp,0xc
 5e8:	6a 05                	push   0x5
 5ea:	e8 b1 ff ff ff       	call   5a0 <sub>
 5ef:	83 c4 10             	add    esp,0x10
```

After the call to `sub`, the stack pointer is corrected, with 16 bytes as we see.


