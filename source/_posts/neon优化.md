---
title: neon优化
date: 2023/5/20 14:14:12
categories: practical-experience
excerpt: 在armv8平台上进行neon优化的一些经验与总结
hide: true
tag: markdown
---
{% note info %}
交叉编译，arm平台环境如下所示
```
Architecture:                    aarch64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
CPU(s):                          2
On-line CPU(s) list:             0,1
Thread(s) per core:              1
Core(s) per socket:              2
Socket(s):                       1
Vendor ID:                       ARM
Model:                           0
Model name:                      Cortex-A72
Stepping:                        r1p0
BogoMIPS:                        400.00
L1d cache:                       64 KiB
L1i cache:                       96 KiB
L2 cache:                        1 MiB
Vulnerability Itlb multihit:     Not affected
Vulnerability L1tf:              Not affected
Vulnerability Mds:               Not affected
Vulnerability Meltdown:          Not affected
Vulnerability Spec store bypass: Vulnerable
Vulnerability Spectre v1:        Mitigation; __user pointer sanitization
Vulnerability Spectre v2:        Not affected
Vulnerability Srbds:             Not affected
Vulnerability Tsx async abort:   Not affected
Flags:                           fp asimd evtstrm aes pmull sha1 sha2 crc32 cpuid
```
{% endnote %} 

## 概述
neon是arm提供的一套SIMD，可以借此实现并行运算，达到提速目的，在C++中`#include<arm_neon.h>`即可使用封装好的`Intrinsics`，[intrinsics参考网站](https://developer.arm.com/architectures/instruction-sets/intrinsics/#f:@navigationhierarchiessimdisa=[Neon]&first=160)

## neon优化实例
1. 2x3矩阵转置，涉及的核心函数为`vtrn1q_f64(A0,A1)`，作用是将A0,A1向量中的奇数位取出(`vtrn2q_f64(A0,A1)`取出偶数位)
```C++
// neon优化的2x3矩阵转置，A是待转置矩阵(按行主序展开)，B是转置结果(行主序)
void neon_2x3_matrix_transpose(const float64_t* A,float64_t* B){
  float64x2_t A0=vld1q_f64(A);
  float64x2_t A1=vld1q_f64(A+3);
  float64x2_t A2=vld1q_f64(A+2);
  float64x2_t A3=vld1q_f64(A+5);

  float64x2_t C0,C1,C2;
  C0=vtrn1q_f64(A0,A1);
  C1=vtrn2q_f64(A0,A1);
  C2=vtrn1q_f64(A2,A3);

  vst1q_f64(B, C0);
  vst1q_f64(B+2, C1);
  vst1q_f64(B+4, C2);
}
```

2. 矩阵乘法
```C++
/*
* 4x4矩阵乘以4x1列向量，矩阵按列主序展开为数组表示
*/
inline void neon_4x4_matrix_4x1_vector_multiply(float64_t* A, float64_t* B, float64_t* C)
{
    float64x2_t A0=vld1q_f64(A);
    float64x2_t A1=vld1q_f64(A+2);
    float64x2_t A2=vld1q_f64(A+4);
    float64x2_t A3=vld1q_f64(A+6);
    float64x2_t A4=vld1q_f64(A+8);
    float64x2_t A5=vld1q_f64(A+10);
    float64x2_t A6=vld1q_f64(A+12);
    float64x2_t A7=vld1q_f64(A+14);

    float64x2_t B0=vld1q_f64(B);
    float64x2_t B1=vld1q_f64(B+2);
    
    float64x2_t C0,C1;
    // set C0 = {0,0}
    C0 = vmovq_n_f64(0);
    C1 = vmovq_n_f64(0);

    // return C0+A0*B0[0]
    C0 = vfmaq_laneq_f64(C0, A0, B0, 0);
    C0 = vfmaq_laneq_f64(C0, A2, B0, 1);
    C0 = vfmaq_laneq_f64(C0, A4, B1, 0);
    C0 = vfmaq_laneq_f64(C0, A6, B1, 1);

    C1 = vfmaq_laneq_f64(C1, A1, B0, 0);
    C1 = vfmaq_laneq_f64(C1, A3, B0, 1);
    C1 = vfmaq_laneq_f64(C1, A5, B1, 0);
    C1 = vfmaq_laneq_f64(C1, A7, B1, 1);
    vst1q_f64(C, C0);
    vst1q_f64(C+2, C1);
}
```