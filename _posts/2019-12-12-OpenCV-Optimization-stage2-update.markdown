---
layout: post
title:  "OpenCV Optimization Practice - Stage II Update"
date:   2019-12-12 12:23:00
categories: SoftwareOptimization
tags: OpenCV
---
* content
{:toc}

# <strong>OpenCV Optimization - Stage II Update </strong>

<p><img src="https://giantpanpan.github.io/img/opencv.png" alt="OpenCV"/></p>

There are the list of tasks I need to add for stage I:
- Perform: Details about the processes of performing optimiation plan
- Safety Test: Will the optimized functions pass the safety test?
- Performace Test: Compare the performance of target functions before and after optimization
- Other Architecture: What is the preformance of the target functions in other platform?







## <strong>Perform Optimization Plan on Aarchie</strong>
After read the source code of fast_math.cpp, I realized that the OpenCV developers call ```lrint(value)``` function inside of the body of ```cvRound()```, but use regular way to perform ```cvCeil()``` and ```cvFloor()``` on Archie

```cpp
CV_INLINE int
cvRound( double value )
{
    ...
#elif defined CV_ICC || defined __GNUC__
    return (int)(lrint(value));
    ...
}
```

```cpp
CV_INLINE int cvCeil( float value )
{
    ...
#else
    int i = (int)value;
    return i + (i < value);
#endif
}
```

<br>
As I mentioned before, the optimization plan is to find functions which are similar with ```lrint(value)``` and see whether the performance of ```cvCeil()``` and ```cvFloor()``` can approach to ```cvRound()```. 


### Looking for similar functions
According to [Linux Programmer's Manual](http://man7.org/linux/man-pages/man3/lrint.3.html), ceil(),ceilf(), floor() and floorf() have similar implementation to lrint(value).

```
long int lrint(double x);
long int lrintf(float x);
```

```
double ceil(double x);
float ceilf(float x);
```

```
double floor(double x);
float floorf(float x);
```

The only concern is that the return value of ```lrint``` is long integer, then use ```(int)lrint(value)```concatinate to integer. But for ceil() and floor() will return double and float, then do the concatinate to int. 


### Code Example after Optimization:

```cpp
CV_INLINE int cvCeil( float value )
{
#if (defined CV__FASTMATH_ENABLE_GCC_MATH_BUILTINS || defined CV__FASTMATH_ENABLE_CLANG_MATH_BUILTINS) \
    && ( \
        defined(__PPC64__) \
    )
    return __builtin_ceilf(value);
#elif __aarch64__
    return (int)(ceilf(value));
#else
    int i = (int)value;
    return i + (i < value);
#endif
}
```

## <strong>Safety Test</strong>
According to the following result, it seems the ```cvCeil``` and ```cvFloor``` pass the safety test. So this optimization will not affect the safety of these two functions. 


```
__aarch64__ cvCeil(float)
total cvCeil float a: 9760000000
correct answer: 9760000000

total cvCeil float b: -3630000000
correct answer: -3630000000
```

```
__aarch64__ cvCeil(double)
total cvCeil double a: 5540000000
correct answer: 5540000000

total cvCeil double b -6340000000
correct answer: -6340000000
```

```
__aarch64__ cvFloor(float)
total cvFloor float a: 9750000000
correct answer: 9750000000

total cvFloor float b: -3640000000
correct answer: -3640000000
```

```
__aarch64__ cvFloor(double)
total cvFloor double a 5530000000
correct answer: 5530000000

total cvFloor double b -6350000000
correct answer: -6350000000
```

## <strong>Performance Test</strong>
Note: these are median results after multiple runs. 

#### Time Before Optimization
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

#### Time After Optimization

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
    <td>0m1.542s</td>
    <td>0m1.534s</td>
  </tr>
  <tr>
    <td>user</td>
    <td>0m1.501s</td>
    <td>0m1.506s</td>
    <td>0m1.501s</td>
  </tr>
  <tr>
    <td>sys</td>
    <td>0m0.032s</td>
    <td>0m0.033s</td>
    <td>0m0.030s</td>
  </tr>


  <tr>
    <th rowspan="3">double</th>
    <td>real</td>
    <td>0m1.535s</td>
    <td>0m1.535s</td>
    <td>0m1.539s</td>
  </tr>
  <tr>
    <td>user</td>
    <td>0m1.503s</td>
    <td>0m1.502s</td>
    <td>0m1.503s</td>
  </tr>
  <tr>
    <td>sys</td>
    <td>0m0.030s</td>
    <td>0m0.030s</td>
    <td>0m0.031s</td>
  </tr>
</table> 

<br>

#### Table of Time Improvment Percentage
 <table style="width:100%">
  <tr>
    <th></th>
    <th></th>
    <th>cvCeil</th>
    <th>cvFloor</th>
  </tr>


  <tr>
    <th rowspan="2">float</th>
    <td>real</td>
    <td>12.14%</td>
    <td>12.64%</td>
  </tr>
  <tr>
    <td>cpu(usr+sys)</td>
    <td>12.21%</td>
    <td>13.16%</td>
  </tr>


  <tr>
    <th rowspan="2">double</th>
    <td>real</td>
    <td>12.54%</td>
    <td>12.46%</td>
  </tr>
  <tr>
    <td>cpu(usr+sys)</td>
    <td>12.61%</td>
    <td>12.59%</td>
  </tr>
</table> 

<br>

#### Usage Before Optimization
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

#### Usage After Optimization
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
    <td>19.64% </td>
    <td>20.56%</td>
  </tr>


  <tr>
    <th>double</th>
    <td>16.88%</td>
    <td>19.71%</td>
    <td>19.28%</td>
  </tr>
</table>


#### <strong>Optimization Result</strong>
According to tables above, we can see that currently the time usage for ```cvRound```, ```cvCeil``` and ```cvFloor``` are very close. The real time for ```cvCeil``` and ```cvFloor``` is about 0.2 second faster. The CPU execution time (usr+sys) of these two function are still approximatey 0.2 second faster. 
The usage after optimization for ```cvCeil``` and ```cvFloor``` decrease around 10% than before and is very close to ```cvRound```

## <strong>Other Architecture</strong>
Aarch64 - Betty

The safety test on Betty is passed. 

<strong>Time Before Optimization</strong>
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

<strong>Time After Optimization</strong>
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
    <td>0m0.562s</td>
    <td>0m0.533</td>
  </tr>
  <tr>
    <td>user</td>
    <td>0m0.548s</td>
    <td>0m0.521s</td>
    <td>0m0.645s</td>
  </tr>
  <tr>
    <td>sys</td>
    <td>0m0.010s</td>
    <td>0m0.011s</td>
    <td>0m0.012s</td>
  </tr>


  <tr>
    <th rowspan="3">double</th>
    <td>real</td>
    <td>0m0.541s</td>
    <td>0m0.560s</td>
    <td>0m0.566s</td>
  </tr>
  <tr>
    <td>user</td>
    <td>0m0.530s</td>
    <td>0m0.555s</td>
    <td>0m0.630s</td>
  </tr>
  <tr>
    <td>sys</td>
    <td>0m0.010s</td>
    <td>0m0.010s</td>
    <td>0m0.010s</td>
  </tr>
</table> 

#### Usage Before Optimization
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

#### Usage After Optimization
 <table style="width:100%">
  <tr>
    <th></th>
    <th>cvRound</th>
    <th>cvCeil</th>
    <th>cvFloor</th>
  </tr>

  <tr>
    <th>float</th>
    <td>7.9%</td>
    <td>4.9% </td>
    <td>8.98%</td>
  </tr>


  <tr>
    <th>double</th>
    <td>7.21%</td>
    <td>6.52%</td>
    <td>7.03%</td>
  </tr>
</table>



## <strong>Conclusion</strong>
Overall, we can see performance imporvement of ```cvCeil``` and ```cvFloor``` in both Archie and Betty. On Archie, the time of these 2 functions droped ~0.2 second and usage dropped ~12% compare with the result before optimization. On Betty, the time for ```cvCeil``` and ```cvFloor``` dropped 0.1 seconds and usage decrease 10% - 50%. Now the time and usage performance of ```cvCeil``` and ```cvFloor``` is very close to  ```cvRound```.  

