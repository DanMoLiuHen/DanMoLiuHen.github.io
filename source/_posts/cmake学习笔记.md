---
title: cmake学习笔记
date: 2022/9/23 14:14:12
categories: learning-notes
excerpt: 本文主要依据cmake官方说明，并加入部分个人理解
hide: false
tag: markdown
---
### 使用
在项目目录下写好cmake文件，然后
```shell
# 常用指令
cmake -B build # 创建一个build目录
cmake --build build  # 在build目录下进行build
```

```makefile
# 指定最小 CMake 版本要求
cmake_minimum_required(VERSION 3.9)
# 设置项目名称
project(answer)

#[[
添加可执行文件 target，类似于原来 Makefile 的：

    answer: main.o answer.o
    main.o: main.cpp answer.hpp
    answer.o: answer.cpp answer.hpp

CMake 会自动找到依赖的头文件，因此不需要特别指定，
当头文件修改的时候，会重新编译依赖它的目标文件。
#]]
add_executable(answer main.cpp answer.cpp)

#[[
使用如下命令构建本项目：

    cmake -B build      # 生成构建目录
    cmake --build build # 执行构建
    ./build/answer      # 运行 answer 程序
#]]
```

```makefile
cmake_minimum_required(VERSION 3.9)
project(answer)

# 添加 libanswer 库目标，STATIC 指定为静态库
add_library(libanswer STATIC answer.cpp)

add_executable(answer main.cpp)

#[[
为 libanswer 库链接 libcurl，这里 PRIVATE 和 PUBLIC 的区别是：
CURL::libcurl 库只会被 libanswer 看到，根级别的 main.cpp 中
无法 include curl 的头文件。
#]]
target_link_libraries(libanswer PRIVATE CURL::libcurl)

#[[
find_package 用于在系统中寻找已经安装的第三方库的头文件和库文件
的位置，并创建一个名为 CURL::libcurl 的库目标，以供链接。
#]]
find_package(CURL REQUIRED)
```

### 简单入门
实现指定C++版本，添加可执行文件等基础功能，并在CMakeLists.txt中定义一个能够在源码里被访问的变量。文件结构如下所示
```
.
├── CMakeLists.txt
├── TutorialConfig.h.in
└── tutorial.cpp
```

CMakeLists.txt中的内容如下所示，主要通过`configure_file(<input> <output>)`实现将变量定义在头文件中以供源码访问

```cmake
# 所有CMakeLists.txt文件都需要指定cmake版本和项目名称
cmake_minimum_required(VERSION 3.10)

# project()调用之后 CMake 定义 Tutorial_VERSION_MAJOR 和 Tutorial_VERSION_MINOR，除了项目名称还添加了版本号
project(Tutorial VERSION 1.0)

# 需要确保在add_executable之前设置C++版本
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# TutorialConfig.h.in 是输入的配置头文件
# 当该句被调用，@Tutorial_VERSION_MAJOR@ 和 @Tutorial_VERSION_MINOR@ 的值将替换为TutorialConfig.h中项目的版本号
# 在tutorial.cpp中应该包含TutorialConfig.h头文件
configure_file(TutorialConfig.h.in TutorialConfig.h)

add_executable(Tutorial tutorial.cpp)
# 使用target_include_directories（）指定可执行目标在哪查找include文件
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
```
如此，在`TutorialConfig.h.in`中定义以下内容，然后在`tutorial.cpp`中`#include "TutorialConfig.h"`即可使用`Tutorial_VERSION_MAJOR`变量和`Tutorial_VERSION_MINOR`变量
```
// the configured options and settings for Tutorial
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```
#### 运行说明
在项目路径下运行以下指令，后续若指令无区别将不再赘述
```
# 指定路径为build文件夹
cmake -B biuld

# 在build路径下进行构建
cmake --build build
```

### 添加library
主要实现添加library，进行链接，使用子目录（subdirectory），条件编译等

文件结构如下所示：
```
├── CMakeLists.txt
├── MathFunctions
│   ├── CMakeLists.txt
│   ├── MathFunctions.cpp
│   ├── MathFunctions.h # 实现mysub两个整数相减
│   ├── myadd.cpp
│   └── myadd.h # 实现myadd两个整数相加
└── tutorial.cpp # 调用子目录下实现函数
```

两个cmakelists.txt文件内容如下所示：
```cmake
# top-level CMakeLists.txt file

cmake_minimum_required(VERSION 3.10)
project(Tutorial VERSION 1.0)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# 添加一个子目录
add_subdirectory(MathFunctions)

add_executable(Tutorial tutorial.cpp)

# 将library链接到可执行目标上（library的名称是由子路径中的cmake文件所指定的）
target_link_libraries(Tutorial PUBLIC MathFunctions)

# 指定库的头文件位置, 将MathFunctions子目录添加为include目录，以便可以找到MathFunctions.h头文件
target_include_directories(Tutorial PUBLIC
                          "${PROJECT_BINARY_DIR}"
                          "${PROJECT_SOURCE_DIR}/MathFunctions"
                          )
```

```cmake
# MathFunctions/CMakeLists.txt

add_library(MathFunctions MathFunctions.cpp)
# 通过option添加一个选项，用户可以修改该选项
option(USE_ADD "Use add" ON)
# USE_MYMATH 为 ON, 编译时USE_MYMATH将被定义，在代码中可以通过该变量是否被定义实现条件编译
if (USE_ADD)
    add_library(SqrtLibrary STATIC
                myadd.cpp)
    target_compile_definitions(MathFunctions PRIVATE "USE_ADD")
    target_link_libraries(MathFunctions PUBLIC SqrtLibrary)
endif()
```

其中`MathFunctions.cpp`文件内容如下所示：
```C++
#include"MathFunctions.h"

// 当USE_ADD为真将调用add方法
#ifdef USE_ADD
#include "myadd.h"
#endif

int mysub(int a,int b){
#ifdef USE_ADD
  return myadd(a,b);
#else
  return a-b;
#endif
}
```
通过上述方法构建出的项目中，使用了静态链接，可以看到在构建出的文件夹内有后缀`.a`的静态库，

#### 运行说明
在项目路径下运行以下指令
```
# 指定USE_ADD为OFF，由于上述文件中已写过默认为ON因此不指定参数USE_ADD将为ON
cmake -DUSE_MYMATH=OFF -B build

cmake --build build
```

### 为一个library添加Usage Requirements
实现自行添加所需的目录。该步骤对应官方教程Step3，在此仅对三种参数做出说明，本人对于该步骤也存在一些疑惑，有待后续补坑

有关三种参数的含义如下所示，其中生产者可以具体为产生library的，消费者是使用library的：
- PRIVATE：仅供producer使用
- PUBLIC：consumer 和producer都使用
- INTERFACE： consumer 使用，producer不使用。面对只有头文件时，无法编译成库，因此只能使用INTERFACE参数

文件结构如下所示，与上一步骤相同：
```
├── CMakeLists.txt
├── MathFunctions
│   ├── CMakeLists.txt
│   ├── MathFunctions.cpp
│   ├── MathFunctions.h # 实现mysub两个整数相减
│   ├── myadd.cpp
│   └── myadd.h # 实现myadd两个整数相加
└── tutorial.cpp # 调用子目录下实现函数
```

两个cmakelists.txt文件内容如下所示：
```cmake
# top-level CMakeLists.txt file

cmake_minimum_required(VERSION 3.10)
project(Tutorial VERSION 1.0)

add_library(tutorial_compiler_flags INTERFACE)
# 使用modern cmake设置C++11 
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)

add_subdirectory(MathFunctions)

list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")

add_executable(Tutorial tutorial.cpp)

target_link_libraries(Tutorial PUBLIC MathFunctions tutorial_compiler_flags)

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
```

```cmake
# MathFunctions/CMakeLists.txt

add_library(MathFunctions MathFunctions.cpp)

option(USE_ADD "Use add" ON)
if (USE_ADD)
    add_library(SqrtLibrary STATIC
                myadd.cpp)
    target_compile_definitions(MathFunctions PRIVATE "USE_ADD")
    target_link_libraries(SqrtLibrary PUBLIC tutorial_compiler_flags)
    target_link_libraries(MathFunctions PUBLIC SqrtLibrary)
endif()

# INTERFACE means things that consumers require but the producer doesn't.
target_include_directories(MathFunctions
                           INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
                           )
```


### 参考资料
[^1]:[Cmake教程](https://cmake.org/cmake/help/latest/guide/tutorial/A%20Basic%20Starting%20Point.html#exercise-1-building-a-basic-project)