---
layout: post
title:  "Software Optimization - SIMD Practice (C Intrinsics)"
date:   2019-10-31 16:49:00
categories: SoftwareOptimization
tags: SIMD gcc Optimization Architecture C AArch64 Assemly-Language
---
* content
{:toc}

## SIMD C Intrinstics
C Intrinsics are function-like extensions to the C language. Although they look like functions, they are compiled inline. Because C Intrinstics are not provided by C language itself, its function is not portable. We need to be very careful if we want to use it. 

The source code for Intrinstic version:
```c++
#include <stdlib.h>
#include <stdio.h>
#include <stdint.h>
#include <arm_neon.h>
#include "vol.h"

int main() {

	int16_t*		data;		// data array
	int16_t*		limit;		// end of input array

	register int16_t*	cursor 		asm("r20");	// array cursor (pointer)
	register int16_t	vol_int		asm("r22");	// volume as int16_t

	int			x;		// array interator
	int			ttl =  0;	// array total

	data=(int16_t*) calloc(SAMPLES, sizeof(int16_t));

	srand(-1);
	printf("Generating sample data.\n");
	for (x = 0; x < SAMPLES; x++) {
		data[x] = (rand()%65536)-32768;
	}

// --------------------------------------------------------------------

	cursor = data; // Pointer to start of array
	limit = data + SAMPLES ;

	vol_int = (int16_t) (0.75 * 32767.0);

	printf("Scaling samples.\n");

	while ( cursor < limit ) {
		// Q: What do these intrinsic functions do? 
		// (See gcc intrinsics documentation)
	
		// A: Signed Saturating Doubling Multiply returning High Half
		// Multiply cursor vector and vol_int vector(a vector with all 
		// vol_int), double the result, then place the highest 16 bits
		// to the cursor. 
		vst1q_s16(cursor, vqdmulhq_s16(vld1q_s16(cursor), vdupq_n_s16(vol_int)));

		// Q: Why is the increment below 8 instead of 16 or some other value?
		// A: In AArch64 Advanced SIMD, there are thirty two 128-bit wide vector register, here is (16 bits)*8(lanes)=128 bits
		// It will execute 8 lanes in parallel per time and then move to next 8 lines. 

		// Q: Why is this line not needed in the inline assembler version
		// Because the "str q0, [%[cursor]],#16" in inline assembler version will do post-increment cursor by 16 bytes
		// of this program?
		cursor += 8;
	}

// --------------------------------------------------------------------

	printf("Summing samples.\n");
	for (x = 0; x < SAMPLES; x++) {
		ttl=(ttl+data[x])%1000;
	}

	// Q: Are the results usable? Are they accurate?
	// A: It is same as the inline version result
	// And its sys time is faster than inline version.
	printf("Result: %d\n", ttl);

	return 0;

}
```
Result for C Intrinstic version
```bash
$ time ./vol_intrinsics 
Generating sample data.
Scaling samples.
Summing samples.
Result: 930

real	0m0.519s
user	0m0.508s
sys	0m0.010s
```