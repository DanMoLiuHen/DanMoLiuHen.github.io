---
title: C++常用库笔记
date: 2023/2/19 14:14:12
categories: learning-notes
excerpt: 总结C++中的部分库如vector,string,cmath,set,list,map使用方式，并尝试复现
hide: false
tag: markdown，C++
---

## vector使用方式
vector初始化方式
```C++
// 只申明
vector<int> v1;
vector<string> v3;

vector<vector<int> >;  //注意空格。初始化二维数组int a[n][n];

vector<int> v5 = { 1,2,3,4,5 }; //使用花括号初始化
vector<string> v6 = { "hi","my","name","is","lee" };
vector<int> v7(5, -1); //申请含5个-1的数组，即-1,-1,-1,-1,-1。

// vector的元素类型是int，初始化为0
// string类型，初始化为""
vector<int> v9(10); 
vector<int> v10(4); 
```

vector赋值方式

## map使用方式
C++中的map与python中的字典比较类似，表现为键值对
map初始化方式
```C++
std:map<int, string> personnel;
```

## cmath使用方式
```C++
exp(x);//返回e的x次方
```