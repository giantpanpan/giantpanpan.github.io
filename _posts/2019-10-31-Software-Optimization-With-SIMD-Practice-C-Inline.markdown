---
layout: post
title:  "Software Optimization - SIMD Practice (C Inline)"
date:   2019-10-31 12:24:00
categories: SoftwareOptimization
tags: SIMD gcc Optimization Architecture C AArch64 Assemly-Language
---
* content
{:toc}

## SIMD C Inline

First of all, let's look at how C Inline assembler works.

The syntax of C Inline is 

```__asm__ ("assembley code template" : outputs : inputs : clobbers);```

It has 4 parameters:
- The assembler template (mandatory)
- Output operands (optional)
- Input operands (optional)
- Clobbers(overwrite) (optional)





## Example:
```c++
int main() {
	int a = 3;
	int b = 19;
	int c;

	// __asm__ ("assembley code template" : outputs : outputs : clobbers)
	__asm__ ("add %0, %1, %2"  //%0 = %1 + %2
        : "=r"(c)              //temp output register %0 value is move to c
        : "r"(a),"r"(b) );     //input value of a, b is placed into temp input register %1, %2 

	printf("%d\n", c);
}
```
For calculating ```c=b%a``` in AArch64 C Inline assemly 
We can use:
```c++
    __asm__("udiv %0, %1, %2" : "=r"(c) : "r"(b), "r"(a) );                //c = b/a = 6
    __asm__("msub %0, %1, %2, %3" : "=r"(c) : "r"(c), "r"(a), "r"(b)  );   //c = 19-(6*3) = 1 
```

This is the quick start of [Arch64 assembly language](https://wiki.cdot.senecacollege.ca/wiki/Aarch64_Register_and_Instruction_Quick_Start) and [C Inline Assembly Instruction](https://gcc.gnu.org/onlinedocs/gcc-4.8.2/gcc/Extended-Asm.html)

## Lab
```c++
// vol_inline.c :: volume scaling in C using AArch64 SIMD
// Chris Tyler 2017.11.29-2019.10.02 - Licensed under GPLv3.
// For the SIMD lab in the Seneca College SPO600 Course

#include <stdlib.h>
#include <stdio.h>
#include <stdint.h>
#include "vol.h"

int main() {

	int16_t*		data;		// input array
	int16_t*		limit;		// end of input array

	// these variables will be used in our assembler code, so we're going
	// to hand-allocate which register they are placed in
	// Q: what is an alternate approach?
    // A: You can make compiler to define a register for your local variable. Not have to do the specification
	
    register int16_t*	cursor 		asm("r20");	// input cursor
	register int16_t	vol_int		asm("r22");	// volume as int16_t

	int			x;		// array interator
	int			ttl =0 ;	// array total

	data=(int16_t*) calloc(SAMPLES, sizeof(int16_t));

	srand(-1);
	printf("Generating sample data.\n");
	for (x = 0; x < SAMPLES; x++) {
		data[x] = (rand()%65536)-32768;
	}

// --------------------------------------------------------------------

	cursor = data;
	limit = data+ SAMPLES ;

	// set vol_int to fixed-point representation of 0.75
	// Q: should we use 32767 or 32768 in next line? why?
    // A: 32768, because the range of 16 bits int is from -32768 to 32767
	vol_int = (int16_t) (0.75 * 32767.0);

	printf("Scaling samples.\n");

	// Q: what does it mean to "duplicate" values in the next line?
    // A: means duplicate vol_int value to a vector register 1 with 8 lanes of 16 bits (half-word) each
	__asm__ ("dup v1.8h,%w0"::"r"(vol_int)); // duplicate vol_int into v1.8h

	while ( cursor < limit ) {
		__asm__ (
			"ldr q0, [%[cursor]], #0 	\n\t"
			// load eight samples into q0 (v0.8h)
			// from in_cursor

			"sqdmulh v0.8h, v0.8h, v1.8h	\n\t"
			// multiply each lane in v0 by v1*2
			// saturate results
			// store upper 16 bits of results into
			// the corresponding lane in v0
		
			// Q: Why is #16 included in the str line
			// but not in the ldr line?	
            // A: Because we need to increment cursor cursor by 16 bits

			"str q0, [%[cursor]],#16		\n\t"
			// store eight samples to [cursor]
			// post-increment cursor by 16 bytes
			// and store back into the pointer register

			// Q: What do these next three lines do?
            // A: 
			: [cursor]"+r"(cursor) //output operand
			: "r"(cursor)          //input operand 
			: "memory"             //clobber, overwrite memory
			);
	}

// --------------------------------------------------------------------

	printf("Summing samples.\n");
	for (x = 0; x < SAMPLES; x++) {
		ttl=(ttl+data[x])%1000;
	}

	// Q: are the results usable? are they correct?
    // A: The result 730, different from vol1

	printf("Result: %d\n", ttl);

	return 0;

}
```

## Result
The result of gcc compiler is 
```bash
$ time ./vol1
Result: -906

real	0m0.477s
user	0m0.446s
sys	0m0.030s
```

C inline is
```bash
$ time ./vol_inline
Generating sample data.
Scaling samples.
Summing samples.
Result: 930

real	0m0.520s
user	0m0.499s
sys	0m0.020s
```

We can see that the User time of gcc compiler is faster than C inline, but situation for sys time is opposite.
