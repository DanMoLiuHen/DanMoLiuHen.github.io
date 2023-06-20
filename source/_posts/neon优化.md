---
title: neon优化
date: 2023/5/20 14:14:12
categories: practical-experience
excerpt: 在armv8平台上进行neon优化的一些经验与总结，使用项目ceres-solver，涉及eigen等库，包括linux下使用perf分析性能
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

## eigen使用
1. lazyProduct works better than .noalias() for matrix-vector
`Aref.transpose().lazyProduct(bref);//Aref matrix, Bref vector`


## perf使用
### 原理
每隔一个固定的时间，在CPU上（每个核上都有）产生一个中断，在中断上看当前是哪个pid，哪个函数，然后给对应的pid和函数加一个统计值，由此可知CPU有百分几的时间在某个pid，或者某个函数上

### 使用perf对程序概览
linux下可以使用perf分析程序性能，执行`perf stat ./your_program`(可能需要sudo权限)，输出程序的性能概览，如下所示：
```bash
            141.56 msec task-clock                #    0.998 CPUs utilized          
                 2      context-switches          #   14.129 /sec                   
                 0      cpu-migrations            #    0.000 /sec                   
             1,537      page-faults               #   10.858 K/sec                  
       488,545,772      cycles                    #    3.451 GHz                    
     1,238,032,409      instructions              #    2.53  insn per cycle         
       179,236,314      branches                  #    1.266 G/sec                  
           620,952      branch-misses             #    0.35% of all branches        

       0.141870608 seconds time elapsed

       0.141916000 seconds user
       0.000000000 seconds sys
```
`sudo perf top -g`可以实时显示当前系统的性能信息

[Modeling Non-linear Least Squares](http://ceres-solver.org/nnls_modeling.html)

上述指标的含义如下：
- task-clock：任务时钟，表示程序运行期间占用了xx的任务时钟周期，该值高，说明程序的多数时间花费在 CPU 计算上而非 IO
- context-switches：上下文切换次数，表示程序执行过程中发生的上下文切换的总次数。上下文切换是指CPU从一个进程或线程切换到另一个进程或线程的操作。
- cpu-migrations：CPU迁移次数，表示程序执行过程中发生的CPU迁移的总次数。用户程序原本在一个CPU上运行，后来迁移到另一个CPU
- page-faults：页错误次数，表示程序执行期间发生的页错误（Page Fault）的总次数。页错误是指程序访问的页面不在主存中，需要从磁盘或其他存储介质加载到主存中的情况。
- Cycles：CPU周期数，表示程序执行期间所消耗的CPU周期数。
- Instructions：指令数，表示程序执行期间的指令数量。
- Branches：分支数，表示程序执行期间发生的总分支数。
- Branch misses：分支未命中次数，表示程序执行期间发生的分支未命中的总次数。

### perf可以监视的事件
指令`sudo perf list`可以列出所有的事件，如下所示：
```
List of pre-defined events (to be used in -e):

  branch-instructions OR branches                    [Hardware event]
  branch-misses                                      [Hardware event]
  bus-cycles                                         [Hardware event]
  cache-misses                                       [Hardware event]
  cache-references                                   [Hardware event]
  cpu-cycles OR cycles                               [Hardware event]
  instructions                                       [Hardware event]
  ref-cycles                                         [Hardware event]

  alignment-faults                                   [Software event]
  bpf-output                                         [Software event]
  cgroup-switches                                    [Software event]
  context-switches OR cs                             [Software event]
  cpu-clock                                          [Software event]
  cpu-migrations OR migrations                       [Software event]
  dummy                                              [Software event]
  emulation-faults                                   [Software event]
  major-faults                                       [Software event]
  minor-faults                                       [Software event]
  page-faults OR faults                              [Software event]
  task-clock                                         [Software event]
  ...
```

### `perf record`记录采集的数据
`sudo perf record -g ./ceres_lba_os`,记录采集的数据,`-g`表示记录程序的调用栈，如果调用栈过多可能显示会出问题，执行之后在目录下会生成一个`perf.data`文件(`perf.data.old`是上一次的执行结果)，该文件需要使用`report`指令打开，比较常用的参数还有`-e`表示记录的事件，如`sudo perf record -e cache-misses -g ./ceres_lba_os`，详细的参数可以使用`perf record --help`查看

`-g`参数作用： 比如fun1()中调用fun11()和fun12()两个函数，没有`-g`统计时单独统计，可能会出现fun11()时间大于fun1()，加上`-g`表示启动堆栈跟踪，在击中fun11()函数时将向上追溯调用栈，即fun11()的耗时会算在fun1()内

`sudo perf report -i perf.data`打开`perf.data`文件，有如下所示内容，`record`缺省事件为`cycles`
```
Samples: 561  of event 'cycles', Event count (approx.): 500263139
  Children      Self  Command       Shared Object            Symbol
+   23.17%     0.00%  ceres_lba_os  [unknown]                [.] 0x000055c220ce6e40
+   20.14%     2.32%  ceres_lba_os  libceres-debug.so.2.1.0  [.] std::_Function_handler<void (int, int), ceres::internal::ProgramEvaluator<ceres::internal::BlockEvaluatePreparer, ceres
+   14.98%     0.00%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::internal::ParallelFor
+   14.62%     7.31%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::internal::ResidualBlock::Evaluate
+   13.19%     0.71%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::internal::SchurEliminator<2, 3, 6>::Eliminate(ceres::internal::BlockSparseMatrixData const&, double const*, doub
     9.80%     9.80%  ceres_lba_os  libm-2.31.so             [.] __sincos
+    9.63%     2.67%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::internal::SchurEliminator<2, 3, 6>::ChunkDiagonalBlockAndGradient
+    7.67%     7.49%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::internal::SchurEliminator<2, 3, 6>::ChunkOuterProduct
+    6.24%     0.00%  ceres_lba_os  [unknown]                [.] 0xb5e79a17e7482e00
+    6.06%     0.00%  ceres_lba_os  [unknown]                [.] 0x0000000000841f0f
+    6.06%     0.00%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::Solver::~Solver
+    6.06%     0.00%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::Solve
+    6.06%     0.00%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::Solver::Solve
+    5.71%     0.00%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::internal::TrustRegionMinimizer::Minimize
+    5.71%     0.00%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::internal::TrustRegionMinimizer::EvaluateGradientAndJacobian
+    5.35%     0.00%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::internal::TrustRegionMinimizer::HandleSuccessfulStep
+    4.28%     4.28%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::internal::BlockSparseMatrix::SquaredColumnNorm
+    3.92%     3.92%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::internal::BlockSparseMatrix::ScaleColumns
+    3.92%     3.92%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::internal::BlockEvaluatePreparer::Prepare
+    3.92%     1.43%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::internal::SchurEliminator<2, 3, 6>::EBlockRowOuterProduct
+    3.92%     3.92%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::internal::MatrixMatrixMultiplyNaive<-1, -1, -1, -1, 0>
     3.74%     3.74%  ceres_lba_os  ceres_lba_os             [.] LocalBAProjectUV::Evaluate
+    3.74%     3.74%  ceres_lba_os  libpthread-2.31.so       [.] __pthread_mutex_unlock
     3.74%     3.57%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::internal::SchurEliminator<2, 3, 6>::BackSubstitute(ceres::internal::BlockSparseMatrixData const&, double const*,
+    3.39%     0.00%  ceres_lba_os  [unknown]                [.] 0000000000000000
+    2.67%     0.36%  ceres_lba_os  libceres-debug.so.2.1.0  [.] ceres::internal::BlockSparseMatrix::RightMultiply
```


`sudo perf diff`对比`perf.data`与`perf.data.old`

## 使用perf优化性能的实例
本示例用于优化cache-misses问题，发生于二重循环中
```
void fun(){
    for(int i=0;i<1e4;i++){
        for(int j=0;j<1e4;j++){
            int a=i+j;
        }
    }
    for(int i=0;i<1e4;i++){
        for(int j=1e4;j<2e4;j++){
            int a=i+j;
        }
    }
    for(int i=0;i<1e4;i++){
        for(int j=2e4;j<3e4;j++){
            int a=i+j;
        }
    }
    for(int i=0;i<1e4;i++){
        for(int j=3e4;j<4e4;j++){
            int a=i+j;
        }
    }
    for(int i=0;i<1e4;i++){
        for(int j=4e4;j<5e4;j++){
            int a=i+j;
        }
    }
    for(int i=0;i<1e4;i++){
        for(int j=5e4;j<6e4;j++){
            int a=i+j;
        }
    }
    for(int i=0;i<1e4;i++){
        for(int j=6e4;j<7e4;j++){
            int a=i+j;
        }
    }
    for(int i=0;i<1e4;i++){
        for(int j=7e4;j<8e4;j++){
            int a=i+j;
        }
    }
    for(int i=0;i<1e4;i++){
        for(int j=8e4;j<9e4;j++){
            int a=i+j;
        }
    }
    for(int i=0;i<1e4;i++){
        for(int j=9e4;j<1e5;j++){
            int a=i+j;
        }
    }
}

void fun2(){
    for(int i=0;i<1e4;i++){
        for(int j=0;j<1e5;j++){
            int a=i+j;
        }
    }
}
```
两个函数都是做二重循环计算，执行的内容和复杂度一致，fun()将fun2()的二重循环拆开（二重循环的内部循环过大，导致计算到内部循环末尾，起始的值已拿出cache，内部循环进入下一次时，重新将起始值放入cache，cache频繁存取影响性能），使用perf查看cache-misses如下所示：
```
Samples: 86  of event 'cache-misses', Event count (approx.): 140933
  Children      Self  Command  Shared Object        Symbol
+   24.87%     1.25%  two      two                  [.] fun2
+   24.21%     0.00%  two      [kernel.kallsyms]    [k] tick_sched_handle.isra.0
+   23.79%     0.42%  two      [kernel.kallsyms]    [k] update_process_times
+   22.25%     0.17%  two      [kernel.kallsyms]    [k] scheduler_tick
+   21.63%     0.00%  two      [kernel.kallsyms]    [k] entry_SYSCALL_64_after_hwframe
+   21.63%     0.00%  two      [kernel.kallsyms]    [k] do_syscall_64
+   18.19%     0.00%  two      two                  [.] fun
```