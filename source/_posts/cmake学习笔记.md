---
title: cmake学习笔记
date: 2022/9/23 14:14:12
categories: learning-notes
excerpt: 主要记录作者学习cmake的笔记，笔记类主要是给作者看，因此访客不必在笔记上花费太多时间阅读理解
hide: false
tag: markdown
---
## 使用
在项目目录下写好cmake文件，然后
```cmd
mkdir build
cd build
cmake ..(若有VS则用指令cmake -G “MinGW Makefiles” ..仅第一次需要使用该指令)
cmake ..
mingw32.make.exe
```