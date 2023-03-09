---
title: C++STL泛型
date: 2023/2/19 14:14:12
categories: learning-notes
excerpt: 总结C++中vector,string,cmath,set,list,map等STL使用方式
hide: false
tag: markdown，C++，C
---
## 注意事项
1. `a.begin() + a.size() == a.end()`a为vector或者string等含有迭代器
2. 示例代码并不是全部代码，只是核心部分

## algorithm
在使用时需要`#include<algorithm>`
### 反转函数
`reverse()`将迭代器区间内的元素反转，可以是string，vector
```C++
//反转[a.begin(),a.end())内的元素
reverse(a.begin(),a.end());
```
### 排序函数
`sort()`排序，默认按照升序排列
```C++
vector<int>a(10);
sort(a.begin(),a.end());
```
也可以自定义排序
```C++
#include<vector>
#include<algorithm>
#include<iostream>
using namespace std;

bool com(int a,int b) {
	return a > b ? true : false;
}
int main() {
	vector<int >a(10);
	for (int i = 0; i < 10; i++)
		a[i] = i;
	sort(a.begin(), a.end(), com);
}
```


## vector使用
### vector初始化方式
```C++
// 只申明
vector<int> v1;
vector<string> v3;

// 初始化二维数组，n1行n2列。默认值为0
vector<vector<int> > stu(n1,vector<int>(n2));  

vector<int> v5 = { 1,2,3,4,5 }; //使用花括号初始化
vector<string> v6 = { "hi","my","name","is","lee" };
vector<int> v7(5, -1); //申请含5个-1的数组，即-1,-1,-1,-1,-1。

// vector的元素类型是int，初始化为0
// string类型，初始化为""
vector<int> v9(10); 
vector<int> v10(4); 
```

### vector元素访问或遍历
1. 下标访问，与数组类似
```C++
//非完整代码
vector<int> a(3);
cout<<a[1];
```
2. 迭代器访问(迭代器类型要与被遍历的vector对象类型一直)
```C++
//非完整代码
vector<int> a(3);
vector<int>::iterator it;
for(it=a.begin();it!=a.end();it++){
  cout<<*it<<" ";
}

```
### 元素插入
`insert()`可以在vector对象的任意位置**前**插入一个元素，插入位置后的所有元素依次向后挪动一个位置，`insert()`要求插入的位置是元素的迭代器而非下标
```C++
//非完整代码
vector<int> a(4);
a.insert(a.begin(),4)//在最前面插入新元素，元素值为4
```
### 元素删除
`erase()`删除vector迭代器所指的一个元素或一个区间内的元素
```C++
vector<int> a(10);
//删除第三个元素（begin()相当于下标访问中的0）
a.erase(a.begin() + 2);
//删除[a.begin()+1,a.begin()+5)元素
a.erase(a.begin() + 1, a.begin() + 5);
``` 
### 其他
`size()`方法返回向量大小（元素个数）
`empty()`返回向量是否为空，非空为0，空为1
```C++
vector<int> a(10);
a.size();
a.empty();
```

---

## string
vector<char>可以处理字符，但不如string方便，有添加，删除，查找和比较的功能，使用需要加`#include<string>`
### 赋值与创建
直接创建缺省为""
```C++
//直接创建
string a;

//赋值
string a="hello";

//字符指针赋给字符对象
char a[20]="hello";
string b=a;
```
### 尾部添加
可以直接用`+`给string对象尾部添加字符或字符串，也可以用`append()`添加字符串（不能是字符）
```C++
//使用+添加
string s;
s+='a';
s+="bf";

//使用append()添加字符串（如果用append()添加字符将会报错）
s.append("123");
```
### 插入字符
`insert()`将字符插入到迭代器位置之**前**
```C++
string s="12345";
s.insert(s.begin()+2,'a');
```
### 删除
1. 清空字符串可以赋空字符串
2. `erase()`与vector类似，删除迭代器所指元素或一个区间内元素
```C++
string a="123456";
a.erase(a.begin(),a.begin()+3);
```
### 返回长度
`length()`返回字符串长度，`size()`也可以返回字符串长度；`empty()`返回字符串是否为空，字符串为空返回1，非空返回0
```C++
string a="1234567";
cout<<a.length()<<" "<<a.size();
cout<<"\n"<<a.empty();
```
### 替换字符
`replace()`可以替换string对象的字符，其重载函数较多
```C++
string s="1234567";
//从第3个开始，将连续的2个字符替换为"nihao"
s.replace(3,2,"nihao");
//输出为123nihao67
```
### 搜索字符
`find()`查找字符串中的第一个字符元素或子串，查到返回下标（从0开始），否则返回4294967295(windows中表现如此，其余未测试)
```C++
string s = "1234567";
cout << s.find('4')<<endl;//返回3
cout << s.find("45")<<endl;//返回3
cout<<s.find('c');//返回4294967295
```
### 比较大小
`compare()`方法可以与其他字符串进行比较，比对方大返回1，比对方小返回-1，相等返回0。
字符串的大小比较原则：逐字符比较，对于字符串A与B比较，若`A[0]>B[0]`返回1，`A[0]<B[0]`返回-1，`A[0]=B[0]`看下一个字符，若下一字符B没有则A大，若下一位A没有则B大
```C++
string s = "1234";
string a = "a";	
cout << s.compare(a);
```
### string对象与字符数组互操作
C语言中使用`printf()`输出时，不能直接输出string对象，需要使用`c_str()`方法
```C++
string s = "1234";
string a = "a";
printf(s.c_str());
```

### string对象与数值相互转化
需要`#include<sstream>`，注意两个自定义的函数
```C++
#include<iostream>
#include<sstream>
#include<string>
using namespace std;
//数值转化为string
string convertToString(double x) {
	ostringstream o; 
	if (o << x) 
		return o.str();
	return "conversion error";//if error
}
//string转化为数值
double convertFromString(const string& s) {
	istringstream i(s); 
	double x; 
	if (i >> x) 
		return x; 
	return 0.0;//if error
}
int main() {
	string s = convertToString(1947.9);
	int p = convertFromString("2048") + 4;
	cout << s << "*\n" << p << endl;

}
```

```C++
/*
* str: 指向要填充的内存块。
* c: 要被设置的值。该值以 int 形式传递，但是函数在填充内存块时是使用该值的无符号字符形式。
* n: 要被设置为该值的字符数。
*/
void *memset(void *str, int c, size_t n)

// 从string对象中取出若干字符存放到数组s中
// s是字符数组，len表示要取出字符的个数，pos表示要取出字符的开始位置。
size_t copy (char* s, size_t len, size_t pos = 0) const;
```

---

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
// 返回x的绝对值
fabs(x)

// 乘方函数，返回a的b次方
pow(a,b)

//返回e的x次方
exp(x);

// 开方，返回根号下x
sqrt(x);
```

## queue使用
先进先出，FIFO，成员函数与栈（stack）类似
```C++
queue<string> q;
//队列加入一个元素
q.push("first");
q.push("second");
cout<<q.front()<<endl;//输出first

//删除队尾元素
q.pop();
cout<<q.front()<<endl;//输出second
```

