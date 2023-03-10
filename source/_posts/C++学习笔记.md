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

1. 迭代器overview，一种迭代器满足位于它上面的迭代器的所有性质，例如双向迭代器满足前向迭代器的所有性质
| 名称       |
|----------|
| 输入迭代器    |
| 前向迭代器    |
| 双向迭代器    |
| 随机访问迭代器  |
| 连续迭代器    |
2. 每种迭代器支持的操作不一致，输入迭代器支持的操作有
|             英              |        中    |
|-----------------------------|------------|
| Dereference Operator (*)    | 解引用运算符（*）  |
| Increment Operator (++)     | 增量运算符（++）  |
| Equal to Operator (==)      | 等于运算符（==）  |
| Not Equal to Operator (!=)  | 不等于运算符（！=） |
