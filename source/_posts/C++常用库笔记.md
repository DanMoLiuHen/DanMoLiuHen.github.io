---
title: C++常用库笔记
date: 2023/2/19 14:14:12
categories: learning-notes
excerpt: 总结C++中的部分库如vector,string,cmath,set,list,map使用方式，并尝试复现
hide: false
tag: markdown，C++
---
## 常用函数
```C++
// 保留几位小数
round()

```

## vector使用方式
vector初始化方式
```C++
// 只申明
vector<int> v1;
vector<string> v3;

// 初始化二维数组，n1行n2列。默认值为0
vector<vector<int>> stu(n1,vector<int>(n2));  

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
map使用迭代器输出
```C++
map<int,int> res;
for (map<int, int>::iterator it = res.begin(); it != res.end(); it++)
  cout << it->first << " " << it->second << endl;;
```

## cmath使用方式
```C++
exp(x);//返回e的x次方

/*
*描述：求浮点数x的绝对值
*
*参数：x：必须是整数或者浮点数
*
*返回值：返回x的绝对值，大于或者等于0
*/
fabs(x)
```

## string
```C++
/*
* str: 指向要填充的内存块。
* c: 要被设置的值。该值以 int 形式传递，但是函数在填充内存块时是使用该值的无符号字符形式。
* n: 要被设置为该值的字符数。
*/
void *memset(void *str, int c, size_t n)

// 将字符串翻转
void reverse(iterator first, iterator last);
reverse(a.first(),a.end())

// 从string对象中取出若干字符存放到数组s中
// s是字符数组，len表示要取出字符的个数，pos表示要取出字符的开始位置。
size_t copy (char* s, size_t len, size_t pos = 0) const;
```