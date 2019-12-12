---
layout: post
title:  "OpenCV Optimization Practice - Stage I Update"
date:   2019-12-09 12:45:00
categories: SoftwareOptimization
tags: OpenCV
---
* content
{:toc}

# <strong>OpenCV Optimization - Stage I Update </strong>

<p><img src="https://giantpanpan.github.io/img/opencv.png" alt="OpenCV"/></p>

After discuss with my instructor, I realized that my OpenCV optimization stage I is not sufficient. 
There are the lisk of tasks I need to add for stage I:
- Test Host - Specify: which test host I will use for benchmaking.
- Test Plan - Safety: What is the result of the test. Is it always returning same&correct result?
- Test Plan - Performance: Display the  detailed performance of the test.
- Optimization Plan: Explan why and how I am going to optimize target functions.
- Other Architecture: What is the preformance of the test in other platform 






## <strong>Build and Banchmark</strong>
### <strong>Test Host: </strong>
The host for initial test is <strong>```Aarch64```</strong> with following features:

```
[wpan17@aarchie ~]$ lscpu
Architecture:        aarch64
Byte Order:          Little Endian
CPU(s):              24
On-line CPU(s) list: 0-23
Thread(s) per core:  1
Core(s) per socket:  2
Socket(s):           12
NUMA node(s):        1
Vendor ID:           ARM
Model:               4
Model name:          Cortex-A53
Stepping:            r0p4
BogoMIPS:            200.00
L1d cache:           32K
L1i cache:           32K
L2 cache:            256K
L3 cache:            4096K
NUMA node0 CPU(s):   0-23
Flags:               fp asimd evtstrm aes pmull sha1 sha2 crc32 cpuid
```

### <strong>Test Data: </strong>
As I mentioned in [Stage I](https://giantpanpan.github.io/2019/11/07/OpenCV-Optimization/), the original testing data is very simple, just need to use cvRound, cvCeil and cvFloor functions for a single float for one million times. But now I realize this test data is not enough. I should try <strong>multiple input</strong>. 


I used 2 pair of test data, one is float and one is double. Each pair contains 2 numbers, one positive and one negative. These numbers are all randomly selected using formula ```(max - min) * ((float)rand() / RAND_MAX) + min;```. The ranges (min - max) are between -1000 to 0 for positive and 0 to 1000 for negative number.

```c++
    float float_a = 975.256165;
    float float_b = -363.565369;
   
    double double_a = 553.034485;
    double double_b = -634.781189;
```


## <strong>Optimization and Testing Plan</strong>
In previous Stage I blog, we already discovered that the performance of ```cvCeil``` and ```cvFloor``` is much worse than ```cvRound```.

Now I need to redo the Benchmark with new test data and plan.

First of all, I create 6 test file for ```cvCeil```, ```cvFloor``` and ```cvRound```.
 <table style="width:100%">
  <tr>
    <th></th>
    <th>cvRound</th>
    <th>cvCeil</th>
    <th>cvFloor</th>
  </tr>
  <tr>
    <th>float</th>
    <td>cvRound_test_f.cpp</td>
    <td>cvCeil_test_f.cpp</td>
    <td>cvFloor_test_f.cpp</td>
  </tr>
  <tr>
    <th>double</th>
    <td>cvRound_test_d.cpp</td>
    <td>cvCeil_test_d.cpp</td>
    <td>cvFloor_test_d.cpp</td>
  </tr>
</table>    

<br>
The main body for these test files are similar like:

```c++
int main(int argc,char** argv){
if( argc != 2 ){
#if __aarch64__
    printf("__aarch64__ cvRound(float)\n");
#endif
    float float_a = 975.256165;
    float float_b = -363.565369;


    long int total_fra = 0; //total number for float
    long int total_frb = 0; //total number for double

    for (int i=0;i<10000000;i++){
    	total_fra += cvRound(975.256165);
	    total_frb += cvRound(-363.565369);
    }

    printf("total cvRound float a: %ld\n",total_fra);
    printf("correct answer: %ld\n\n",(long int)round(float_a)*10000000); 

    printf("total cvRound float b: %ld\n", total_frb);
    printf("correct answer: %ld\n\n",(long int)round(float_b)*10000000);

    return 0;
    }
}
```

### <strong>Test Plan</strong>
In above script, we will call ```cvRound()``` function one million time with both positive and negative float. The result of will be added up to long integer total_fra and total_frb and will be printed out. The expected result will be calculated use regular ```round()``` function. The test plan for other two functions will follow the same method and same test data.

#### <strong>Test Plan - Safety</strong>
cvRound_test_f.cpp:
```c++
total cvRound float a: 9750000000
correct answer: 9750000000

total cvRound float b: -3640000000
correct answer: -3640000000
```

cvRound_test_d.cpp:
```c++
total cvRound double a: 5530000000
correct answer: 5530000000

total cvRound double b: -6350000000
correct answer: -6350000000
```

cvCeil_test_f.cpp:
```c++
total cvCeil float a: 9760000000
correct answer: 9760000000

total cvCeil float b: -3630000000
correct answer: -3630000000
```

cvCeil_test_d.cpp:
```c++
total cvCeil double a: 5540000000
correct answer: 5540000000

total cvCeil double b -6340000000
correct answer: -6340000000
```

cvFloor_test_f.cpp
```c++
total cvFloor float a: 9750000000
correct answer: 9750000000

total cvFloor float b: -3640000000
correct answer: -3640000000
```

cvFloor_test_d.cpp
```c++
total cvFloor double a 5530000000
correct answer: 5530000000

total cvFloor double b -6350000000
correct answer: -6350000000
```

So far the result of thse three functions are all correct. After we modify the ```cvCeil ``` and ```cvFloor``` in next stage, this result will be used as an reference to check whether the optimization implementation is safe. 

#### <strong>Test Plan - Performance</strong>

<strong>Time Usage with same test condition</strong>
 <table style="width:100%">
  <tr>
    <th></th>
    <th></th>
    <th>cvRound</th>
    <th>cvCeil</th>
    <th>cvFloor</th>
  </tr>


  <tr>
    <th rowspan="3">float</th>
    <td>real</td>
    <td>0m1.535s</td>
    <td>0m1.755s</td>
    <td>0m1.756s</td>
  </tr>
  <tr>
    <td>user</td>
    <td>0m1.501s</td>
    <td>0m1.702</td>
    <td>0m1.712s</td>
  </tr>
  <tr>
    <td>sys</td>
    <td>0m0.032s</td>
    <td>0m0.051</td>
    <td>0m0.041s</td>
  </tr>


  <tr>
    <th rowspan="3">double</th>
    <td>real</td>
    <td>0m1.535s</td>
    <td>0m1.755s</td>
    <td>0m1.758s</td>
  </tr>
  <tr>
    <td>user</td>
    <td>0m1.503s</td>
    <td>0m1.712s</td>
    <td>0m1.704s</td>
  </tr>
  <tr>
    <td>sys</td>
    <td>0m0.030s</td>
    <td>0m0.041s</td>
    <td>0m0.051s</td>
  </tr>
</table>  

 <br>   
<strong>Perf with same test conditions</strong>

The perf report of shows us the runtime usage percentage of cvRound and other functions:
```
  57.58%  cvRound_float  libc-2.27.so                  [.] _mcount@@GLIBC_2.18
  11.77%  cvRound_float  cvRound_float                 [.] main
  10.92%  cvRound_float  cvRound_float                 [.] cvRound
   7.32%  cvRound_float  libm-2.27.so                  [.] __lrint
```

```
  59.63%  cvRound_double  libc-2.27.so                [.] _mcount@@GLIBC_2.18
  11.11%  cvRound_double  cvRound_double              [.] main
  10.29%  cvRound_double  cvRound_double              [.] cvRound
   6.59%  cvRound_double  libm-2.27.so                [.] __lrint
```

Compare with cvRound, we can see that the percentage of cvCeil and cvFloor is much higher:
```
  51.79%  cvCeil_float  libc-2.27.so                [.] _mcount@@GLIBC_2.18
  30.13%  cvCeil_float  cvCeil_float                [.] cvCeil
   9.49%  cvCeil_float  cvCeil_float                [.] main
   3.82%  cvCeil_float  ld-2.27.so                  [.] do_lookup_x
```

```
  50.39%  cvCeil_double  libc-2.27.so                [.] _mcount@@GLIBC_2.18
  31.32%  cvCeil_double  cvCeil_double               [.] cvCeil
   9.81%  cvCeil_double  cvCeil_double               [.] main
   3.81%  cvCeil_double  ld-2.27.so                  [.] do_lookup_x
```

```
  50.91%  cvFloor_float  libc-2.27.so                  [.] _mcount@@GLIBC_2.18
  30.07%  cvFloor_float  cvFloor_float                 [.] cvFloor
  10.33%  cvFloor_float  cvFloor_float                 [.] main
   3.62%  cvFloor_float  ld-2.27.so                    [.] do_lookup_x
```

```
  49.91%  cvFloor_double  libc-2.27.so                [.] _mcount@@GLIBC_2.18
  31.29%  cvFloor_double  cvFloor_double              [.] cvFloor
  10.20%  cvFloor_double  cvFloor_double              [.] main
   3.78%  cvFloor_double  ld-2.27.so                  [.] do_lookup_x
```

<br>
<strong>Usage Table</strong>
 <table style="width:100%">
  <tr>
    <th></th>
    <th>cvRound</th>
    <th>cvCeil</th>
    <th>cvFloor</th>
  </tr>

  <tr>
    <th>float</th>
    <td>18.24%</td>
    <td>30.13% </td>
    <td>30.07%</td>
  </tr>


  <tr>
    <th>double</th>
    <td>16.88%</td>
    <td>31.32%</td>
    <td>31.29%</td>
  </tr>
</table>

<br>
According to the <strong>time</strong> and <strong>perf</strong> result, we can see that the performance of ```cvCeil``` and ```cvFloor``` is much worse than ```cvRound```. There should be some room for us to to improve ```cvCeil``` and ```cvFloor```, to make their performance approach to ```cvRound```. 

#### <strong>Optimization Plan</strong>
After looking into these three function, I realize that the reason why cvRound performs better is that it will call a special GNUC function ```lrint()``` inside of its body. And ```lrint()``` is an optimized function which will round the input to the nearest integer.

```
       │    _ZL7cvRoundd():
 12.91 │      stp    x29, x30, [sp, #-32]!
 11.98 │      mov    x29, sp
       │      mov    x0, x30
 26.16 │      str    d0, [sp, #24]
       │    → bl     _mcount@plt
 11.80 │      ldr    d0, [sp, #24]
       │    → bl     lrint@plt
 11.80 │      ldp    x29, x30, [sp], #32
 25.35 │    ← ret
``` 

```
       │   0000000000044218 <lrint>:                                           
       │   lrint():                                                            
 20.48 │     frintx d0, d0                                                     
 79.52 │     fcvtzs x0, d0                                                     
       │   ← ret  
```

But for ```cvCeil()``` and ```cvRound```, there is no such optimization

```
Percent│    000000000040692c <cvCeil(double)>:                                 
       │    _ZL6cvCeild():                                                     
  4.13 │      stp    x29, x30, [sp, #-48]!                                     
  4.37 │      mov    x29, sp                                                   
       │      mov    x0, x30                                                   
  7.31 │      str    d0, [sp, #24]                                             
       │    → bl     _mcount@plt                                               
  4.12 │      ldr    d0, [sp, #24]                                             
 12.34 │      fcvtzs w0, d0                                                    
  3.79 │      str    w0, [sp, #44]                                             
  3.56 │      ldr    w0, [sp, #44]                                             
 14.57 │      scvtf  d0, w0                                                    
       │      ldr    d1, [sp, #24]                                             
 15.80 │      fcmpe  d1, d0                                                    
  4.37 │      cset   w0, gt                                                    
  3.70 │      and    w0, w0, #0xff                                             
  3.99 │      mov    w1, w0                                                    
       │      ldr    w0, [sp, #44]                                             
  7.45 │      add    w0, w1, w0                                                
  3.42 │      ldp    x29, x30, [sp], #48                                       
  7.07 │    ← ret 
```

<br>
<strong> So the test plan is to find optimized GNUC funtions which are similar with ```lrint()``` for ```cvCeil``` and ```cvFloor```, to imporve the performance of these two functions. </strong>

## <strong>Other Architecture</strong>
Performence Comparison between different architecture with same test condition

#### <strong>Aarch64 - Betty</strong>
```
[wpan17@bbetty test]$ lscpu
Architecture:        aarch64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              8
On-line CPU(s) list: 0-7
Thread(s) per core:  1
Core(s) per socket:  2
Socket(s):           4
NUMA node(s):        1
Vendor ID:           ARM
Model:               2
Model name:          Cortex-A57
Stepping:            r1p2
BogoMIPS:            500.00
NUMA node0 CPU(s):   0-7
Flags:               fp asimd evtstrm aes pmull sha1 sha2 crc32 cpuid
```

<br>
<strong>Time</strong>
 <table style="width:100%">
  <tr>
    <th></th>
    <th></th>
    <th>cvRound</th>
    <th>cvCeil</th>
    <th>cvFloor</th>
  </tr>


  <tr>
    <th rowspan="3">float</th>
    <td>real</td>
    <td>0m0.559s</td>
    <td>0m0.654s</td>
    <td>0m0.656s</td>
  </tr>
  <tr>
    <td>user</td>
    <td>0m0.548s</td>
    <td>0m0.633s</td>
    <td>0m0.645s</td>
  </tr>
  <tr>
    <td>sys</td>
    <td>0m0.010s</td>
    <td>0m0.020s</td>
    <td>0m0.010s</td>
  </tr>


  <tr>
    <th rowspan="3">double</th>
    <td>real</td>
    <td>0m0.541s</td>
    <td>0m0.655s</td>
    <td>0m0.651s</td>
  </tr>
  <tr>
    <td>user</td>
    <td>0m0.530s</td>
    <td>0m0.634s</td>
    <td>0m0.630s</td>
  </tr>
  <tr>
    <td>sys</td>
    <td>0m0.010s</td>
    <td>0m0.020s</td>
    <td>0m0.021s</td>
  </tr>
</table>  

<br>
<strong>Usage</strong>
 <table style="width:100%">
  <tr>
    <th></th>
    <th>cvRound</th>
    <th>cvCeil</th>
    <th>cvFloor</th>
  </tr>

  <tr>
    <th>float</th>
    <td>7.79%</td>
    <td>9.33% </td>
    <td>9.86%</td>
  </tr>


  <tr>
    <th>double</th>
    <td>7.68%</td>
    <td>12.61%</td>
    <td>10.83%</td>
  </tr>
</table>

<br>
The performance of these three functions on Betty seems much better than Archie. But sill, the performence of cvRound is still better than other two functions.

#### <strong>x86_64 xerxes</strong>
I plan to use run the same test on xerxes, but there is no engough space for me to install OpenCV on it.
```
[wpan17@xerxes project]$ git clone https://github.com/opencv/opencv.git
Cloning into 'opencv'...
remote: Enumerating objects: 7, done.
remote: Counting objects: 100% (7/7), done.
remote: Compressing objects: 100% (7/7), done.
fatal: write error: No space left on device55 MiB | 12.94 MiB/s   
fatal: index-pack failed
```

