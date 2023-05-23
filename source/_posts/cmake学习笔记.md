---
title: cmake学习笔记
date: 2022/9/23 14:14:12
categories: learning-notes
excerpt: 主要记录cmake学习笔记，似乎写着写着就会了，专门学反而不会。
hide: false
tag: markdown
---
## 使用
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