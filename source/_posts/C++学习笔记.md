---
title: C++学习笔记
date: 2023/3/3 19:02:23
categories: learning-notes
excerpt: 记录C++中的细枝末节
hide: true
tag: 
- markdown
- C++
---
函数后面加const表示函数不可以修改class的成员
## 动态申请
申请与释放一维数组
```C++
//一维数组
int *a=new int[10];
delete[] a;

//二维数组(申请10行9列二维数组)
int **a=new int*[10];
for(int i=0;i<10;i++)
  a[i]=new int[9];

for(int i=0;i<10;i++)
  delete[] a[i];
delete[] a;
```

## 析构函数
1. 析构函数没有参数，没有返回值，不能重载。
2. 虚构函数必须是public权限
3. 纯虚析构函数必须有定义
```C++
class test3 {
public:
	virtual  ~test3() = 0;
};
test3::~test3() { cout << "test3" << endl; }

class test4 :public test3 {
public:
	~test4() { cout << "test4" << endl; }
};

int main() {
	test4 t;
	return 0;
}
//输出：
//test4
//test3
//缺少~test3()实现则会报错
```

## 迭代器
1. 共有以下几种迭代器，每种迭代器具有的权限不同，能执行的操作也不同。**除输出迭代器**一种迭代器满足位于它上面的迭代器的所有性质，例如双向迭代器满足前向迭代器的所有性质
| 名称       |
|----------|
| 输入迭代器    |
| 输出迭代器    |
| 前向迭代器    |
| 双向迭代器    |
| 随机访问迭代器  |
| 连续迭代器    |
2. 输入迭代器(Input Iterator)可以遍历容器的元素，但不能提供修改容器的数据值的访问权限。其支持的操作有
|             英              |        中    |
|-----------------------------|------------|
| Dereference Operator (*)    | 解引用运算符（*）  |
| Increment Operator (++)     | 增量运算符（++）  |
| Equal to Operator (==)      | 等于运算符（==）  |
| Not Equal to Operator (!=)  | 不等于运算符（！=） |
3. 输出迭代器(Output Iterator)与输入迭代器是互补的，即在功能上彼此相反。可用于分配和操作容器数据项的值。但不能访问或遍历容器的元素。其支持的操作有
|             英              |        中    |
|-----------------------------|------------|
| Dereference Operator (* , ->)    | 解引用运算符（*，->）  |
| Increment Operator (++)     | 增量运算符（++）  |
| Equal to Operator (==)      | 等于运算符（==）  |
| Not Equal to Operator (!=)  | 不等于运算符（！=） |
4. 前向迭代器（Forward Iterator）可以看做输入和输出迭代器的组合。可以修改容器的数据和遍历元素，但前向迭代器不允许反向或向后遍历元素，其支持的操作有
|             英              |        中    |
|-----------------------------|------------|
| Dereference Operator (* , ->)    | 解引用运算符（*，->）  |
| Increment Operator (++)     | 增量运算符（++）  |
| Equal to Operator (==)      | 等于运算符（==）  |
| Not Equal to Operator (!=)  | 不等于运算符（！=） |
5. 双向迭代器(Bi-directional Iterator)与输入迭代器非常相似。唯一的区别是，双向迭代器可以在两个方向遍历元素。其支持的操作有
|             英              |        中    |
|-----------------------------|------------|
| Dereference Operator (* , ->)    | 解引用运算符（*，->）  |
| Increment Operator (++)     | 增量运算符（++）  |
| Equal to Operator (==)      | 等于运算符（==）  |
| Not Equal to Operator (!=)  | 不等于运算符（！=） |
| Decrement Operator (–)  | 减量运算符（–） |
6. 随机访问迭代器（Random-Access Iterator），可以在容器中的任何索引访问数据项。包含双向迭代器的所有属性。可以访问指针减法和加法 

## 数据类型
有三种数据类型
1. 整型：short、int、long
2. 浮点型：float、double
3. 字符类型：char
```C++
#ifndef __int8_t_defined  
# define __int8_t_defined  
typedef signed char             int8_t;   
typedef short int               int16_t;  
typedef int                     int32_t;  
# if __WORDSIZE == 64  
typedef long int                int64_t;  
# else  
__extension__  
typedef long long int           int64_t;  
# endif  
#endif  
  
  
typedef unsigned char           uint8_t;  
typedef unsigned short int      uint16_t;  
#ifndef __uint32_t_defined  
typedef unsigned int            uint32_t;  
# define __uint32_t_defined  
#endif  
#if __WORDSIZE == 64  
typedef unsigned long int       uint64_t;  
#else  
__extension__  
typedef unsigned long long int  uint64_t;  
#endif  
```