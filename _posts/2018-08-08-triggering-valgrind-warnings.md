---
layout: post
title:  "Triggering Valgrind warning"
date:   2018-08-08 17:58:00 +0100
categories: 
---

# Triggering Valgrind warnings

As I was playing with Valgrind's memcheck, I wanted to create a small executable which does not output any warnings when compiled with GCC, but triggers on all "categories" that Valgrind should detect.

This is the source code of the program:

```
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

/* This little program will output no warnings by the GCC compiler, 
 * but will trigger a lot of errors when instrumented with valgrind.
 * */

int main(void) {
  // overrun top of the stack
  int a;
  int b = *(&a - 0x100);

  // underrun heap block
  char *c = malloc(10);
  char d = c[-1];
  // overrun heap block
  char e = c[10];

  // access memory after it is freed
  free(c);
  char f = c[0];

  // accessing undefined values
  char *g = malloc(10);
  printf("g[0] = %d\n", g[0]);

  // incorrect freeing of heap memory (e.g. double free)
  free(g);
  free(g);

  // Overlapping src and dst pointers in memcpy and related functions.
  char *h = malloc(24);
  memset(h, 'A', 23);
  h[23] = 0;
  char *i = h + 8;
  strcpy(h, i);
  free(h);

  // Fishy arguments
  int j = -3;
  void *k = malloc(j);
  free(k);

  // Memory leak
  void *l = malloc(16);

  return 0;
}
```

Next, I compile the program with debugging symbols, as to have more information in Valgrind on the precise location of the bug in the source files.

```
gcc -g -o a a.c
```

Now run Valgrind:

```
valgrind ./a
```

and you see all kind of errors:

```
$ valgrind ./a
==5892== Memcheck, a memory error detector
==5892== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==5892== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==5892== Command: ./a
==5892== 
==5892== Invalid read of size 4
==5892==    at 0x40061E: main (a.c:12)
==5892==  Address 0x1ffefff1e4 is on thread 1's stack
==5892==  1020 bytes below stack pointer
==5892== 
==5892== Invalid read of size 1
==5892==    at 0x400639: main (a.c:16)
==5892==  Address 0x51fa03f is 1 bytes before a block of size 10 alloc'd
==5892==    at 0x4C2EBAB: malloc (vg_replace_malloc.c:299)
==5892==    by 0x400630: main (a.c:15)
==5892== 
==5892== Invalid read of size 1
==5892==    at 0x400644: main (a.c:18)
==5892==  Address 0x51fa04a is 0 bytes after a block of size 10 alloc'd
==5892==    at 0x4C2EBAB: malloc (vg_replace_malloc.c:299)
==5892==    by 0x400630: main (a.c:15)
==5892== 
==5892== Invalid read of size 1
==5892==    at 0x40065B: main (a.c:22)
==5892==  Address 0x51fa040 is 0 bytes inside a block of size 10 free'd
==5892==    at 0x4C2FDAC: free (vg_replace_malloc.c:530)
==5892==    by 0x400656: main (a.c:21)
==5892==  Block was alloc'd at
==5892==    at 0x4C2EBAB: malloc (vg_replace_malloc.c:299)
==5892==    by 0x400630: main (a.c:15)
==5892== 
==5892== Conditional jump or move depends on uninitialised value(s)
==5892==    at 0x4E8B4A9: vfprintf (in /usr/lib64/libc-2.27.so)
==5892==    by 0x4E935F5: printf (in /usr/lib64/libc-2.27.so)
==5892==    by 0x400689: main (a.c:26)
==5892== 
==5892== Use of uninitialised value of size 8
==5892==    at 0x4E8792E: _itoa_word (in /usr/lib64/libc-2.27.so)
==5892==    by 0x4E8B225: vfprintf (in /usr/lib64/libc-2.27.so)
==5892==    by 0x4E935F5: printf (in /usr/lib64/libc-2.27.so)
==5892==    by 0x400689: main (a.c:26)
==5892== 
==5892== Conditional jump or move depends on uninitialised value(s)
==5892==    at 0x4E87939: _itoa_word (in /usr/lib64/libc-2.27.so)
==5892==    by 0x4E8B225: vfprintf (in /usr/lib64/libc-2.27.so)
==5892==    by 0x4E935F5: printf (in /usr/lib64/libc-2.27.so)
==5892==    by 0x400689: main (a.c:26)
==5892== 
==5892== Conditional jump or move depends on uninitialised value(s)
==5892==    at 0x4E8C0B8: vfprintf (in /usr/lib64/libc-2.27.so)
==5892==    by 0x4E935F5: printf (in /usr/lib64/libc-2.27.so)
==5892==    by 0x400689: main (a.c:26)
==5892== 
==5892== Conditional jump or move depends on uninitialised value(s)
==5892==    at 0x4E8B379: vfprintf (in /usr/lib64/libc-2.27.so)
==5892==    by 0x4E935F5: printf (in /usr/lib64/libc-2.27.so)
==5892==    by 0x400689: main (a.c:26)
==5892== 
g[0] = 0
==5892== Invalid free() / delete / delete[] / realloc()
==5892==    at 0x4C2FDAC: free (vg_replace_malloc.c:530)
==5892==    by 0x4006A1: main (a.c:30)
==5892==  Address 0x51fa090 is 0 bytes inside a block of size 10 free'd
==5892==    at 0x4C2FDAC: free (vg_replace_malloc.c:530)
==5892==    by 0x400695: main (a.c:29)
==5892==  Block was alloc'd at
==5892==    at 0x4C2EBAB: malloc (vg_replace_malloc.c:299)
==5892==    by 0x40066A: main (a.c:25)
==5892== 
==5892== Source and destination overlap in strcpy(0x51fa520, 0x51fa528)
==5892==    at 0x4C31E10: strcpy (vg_replace_strmem.c:510)
==5892==    by 0x4006EF: main (a.c:37)
==5892== 
==5892== Argument 'size' of function malloc has a fishy (possibly negative) value: -3
==5892==    at 0x4C2EBAB: malloc (vg_replace_malloc.c:299)
==5892==    by 0x40070F: main (a.c:42)
==5892== 
==5892== 
==5892== HEAP SUMMARY:
==5892==     in use at exit: 16 bytes in 1 blocks
==5892==   total heap usage: 5 allocs, 5 frees, 1,084 bytes allocated
==5892== 
==5892== LEAK SUMMARY:
==5892==    definitely lost: 16 bytes in 1 blocks
==5892==    indirectly lost: 0 bytes in 0 blocks
==5892==      possibly lost: 0 bytes in 0 blocks
==5892==    still reachable: 0 bytes in 0 blocks
==5892==         suppressed: 0 bytes in 0 blocks
==5892== Rerun with --leak-check=full to see details of leaked memory
==5892== 
==5892== For counts of detected and suppressed errors, rerun with: -v
==5892== Use --track-origins=yes to see where uninitialised values come from
==5892== ERROR SUMMARY: 12 errors from 12 contexts (suppressed: 0 from 0)
```

*Some interesting fact:* Using `memcpy` instead of `strcpy` did not trigger the memory overlap warning, when compiled for 64-bit, but did when compiled for 32-bit. If I understood the info in the Valgrind's mailing lists correctly, this would have to do with `memcpy` being called from a different library on Fedora.
