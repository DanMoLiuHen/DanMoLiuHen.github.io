---
title: C++STL泛型
date: 2023/2/19 14:14:12
categories: learning-notes
excerpt: 总结C++中vector,string,set,list,map等STL使用方式，适合于竞赛了解用法，没有深入
hide: false
tag: 
- markdown
- C++
- C
---
{% note warning %}
1. `a.begin() + a.size() == a.end()`a为vector或者string等含有迭代器
2. 示例代码并不是全部代码，只是核心部分
3. 忽略某个警告，如忽略4305警告`#pragma warning(disable:4305)`
4. 有关迭代器的相关内容在C++STL泛型进阶中会有详细说明
5. 强烈建议把cppreference作为字典使用，非常方便
{% endnote %} 

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
### 二分查找函数
在 log(n) 的时间复杂度内查找给定值
1. `lower_bound()`在指定区域内查找**不小于目标值**的第一个元素。
```C++
// 在 [first, last) 区域内查找第一个不小于目标值的元素。
ForwardIterator lower_bound (ForwardIterator first, ForwardIterator last,const T& val);
// 在 [first, last) 区域内查找第一个不符合 comp 规则的元素
ForwardIterator lower_bound (ForwardIterator first, ForwardIterator last,const T& val, Compare comp);
```
2. `upper_bound()`在指定区域内查找**大于目标值**的第一个元素。
```C++
//查找[first, last)区域中第一个大于 val 的元素。
ForwardIterator upper_bound (ForwardIterator first, ForwardIterator last,const T& val);
//查找[first, last)区域中第一个不符合 comp 规则的元素
ForwardIterator upper_bound (ForwardIterator first, ForwardIterator last,const T& val, Compare comp);
```
3. 如果在序列中有多个（>=1）等于给定值的元素，`lower_bound`函数返回其中等于给定值的第一个迭代器，而`upper_bound`函数返回的是最后一个等于给定值的元素的下一个元素迭代器。如果给定值在序列中不存在，`lower_bound`函数与`upper_bound`函数返回一致，是指向第一个大于该值的元素的迭代器。
```C++
vector<int>a = { 1,9,12,30,32,45,174,1345 };
auto it = upper_bound(a.begin(), a.end(), 30);
auto it2 = lower_bound(a.begin(), a.end(), 30);
cout << *it << endl;
cout << *it2 << endl;
// 输出32\n30
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
`vector<char>`可以处理字符，但不如string方便，有添加，删除，查找和比较的功能，使用需要加`#include<string>`
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
可以直接用`+`给string对象尾部添加**字符或字符串**，也可以用`append()`添加字符串（不能是字符）
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
### 查找/搜索字符
`find()`查找字符串中的第一个字符元素或子串，查到返回下标（从0开始），否则返回4,294,967,295(2^32-1,windows中表现如此，其余未测试)
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
C语言中使用`printf()`输出时，不能直接输出string对象，需要使用`c_str()`方法，将string对象转为了const char*
```C++
string s = "1234";
string a = "a";
printf(s.c_str());
```
### 其他
#### 1. 截取子串
`string substr(int pos = 0,int n = n) const;`返回pos开始（包含pos）的n个字符组成的字符串

2. 分割字符串
在STL中
```C++
vector<string> mysplit(string a,const string& b) {
	vector<string>res;
	for (int tmp = a.find(b);tmp<a.size();tmp=a.find(b)) {
		res.emplace_back(a.substr(0, tmp));
		a = a.substr(tmp + b.size(), a.size());
	}
	res.emplace_back(a);
	return res;
}
```

---

## set使用
- set集合容器实现了红黑树（Red-Black Tree）的平衡二叉检索树的数据结构，每个子树根节点键值大于左子树所有节点的键值，而小于右子树所有节点的键值，并且根节点左子树高度与右子树高度相同，**不会重复插入相同键值元素**。
- 平衡二叉检索树的检索使用中序遍历，效率高于vector,list,deque等容器
- set中的键值不能直接修改。一旦修改将根据新的键值旋转子树，以达到平衡，因此修改的键值可能不在原位置，构建set集合主要是为了快速检索
- multiset,map,multimap的内部结构也是平衡二叉检索树
- 使用需要`#include<set>`
### 创建
与其他容器类似，元素的排列按默认的比较规则（由小到大，在比较规则函数未自定义时）
```C++
set<int> a;
```
### 插入与中序遍历
`insert()`将元素插入集合，默认是按照由小到大插入，使用前向迭代器对集合中序遍历，结果即为排序结果
```C++
set<int>a;
a.insert(4);
a.insert(2);
a.insert(9);
a.insert(2);//第二次插入无效
for (set<int>::iterator it = a.begin(); it != a.end(); it++) {
  cout << *it << " ";
}
//输出为2 4 9
```
### 反向遍历
反向迭代器`reverse_iterator`能够反向遍历集合，输出为集合反向排序结果，其中`rbegin()`和`rend()`两个方法，分别给出反向遍历的开始位置和结束位置
```C++
set<int>a;
a.insert(4);
a.insert(2);
a.insert(9);
a.insert(2);
for (set<int>::reverse_iterator it = a.rbegin(); it != a.rend(); it++) {
  cout << *it << " ";
}
```
### 元素删除
`erase()`方法可以删除某个迭代器位置上的元素，等于某键值的元素，一个区间上的元素和清空集合。删除的效率也比较高，同时自动调整内部的红黑树平衡
```C++
set<int>a;
a.insert(4);
a.insert(2);
a.insert(9);
a.insert(8);
a.erase(2);//删除键值为2的元素
for (set<int>::reverse_iterator it = a.rbegin(); it != a.rend(); it++) {
  cout << *it << " ";
}
cout << endl;
a.erase(a.begin());//删除第一个元素(键值最小的元素)
for (set<int>::iterator it = a.begin(); it != a.end(); it++) {
  cout << *it << " ";
}
```
### 元素检索
`find()`方法（不是算法函数`find()`，算法函数时间复杂度为线性即逐个比较）对集合进行搜索，时间复杂度为`logn`，如果查到键值，返回该键值的迭代器位置，否则，返回集合最后一个元素后面的一个位置，即`end()`。
```C++
set<int>a;
a.insert(4);
a.insert(2);
a.insert(9);
a.insert(8);
auto it = a.find(4);//返回键值为4的迭代器位置
if (it != a.end())
  cout << "find ";
```
### 自定义比较函数
在插入操作`insert()`中集合根据比较函数放置元素，缺省按照由小到大的顺序，也可以自己编写，有两种编写比较函数的方法：
1. 元素不是结构体。写一个结构体然后重载`()`操作符(相当于是一个仿函数，有关仿函数介绍可见C++STL泛型进阶)
```C++
#include <set> 
#include <iostream>
using namespace std; 
//自定义比较函数 myComp，重载“()”操作符 
struct myComp {
	bool operator()(const int& a, const int& b)const {//形参类型int必须与set中的值类型一致
		if (a != b) // 取等时需要特殊处理，否则小于等于的情况混杂
			return a > b;
		else
			return a > b;
	}
};
int main() {
	set<int,myComp> s; //第二个类型必须是结构体
	s.insert(1); 
	s.insert(12); 
	s.insert(6); 
	s.insert(8);
	set<int,myComp>::iterator it;
	for (it = s.begin(); it != s.end(); it++) {
		cout << *it << " ";
	}
}
```
2. 元素是结构体。将比较函数写在结构体内，重载`<`符号自定义排序规则
```C++
#include <set> 
#include <iostream>
using namespace std; 
struct Info {
	string name;
	float score;
	Info(string n, float s) :name(n), score(s) {}
	bool operator<(const Info &a) const {//形参类型int必须与set中的值类型一致
		//按照score从大到小排序
		return a.score < score;
	}
};
int main() {
	set<Info> s; //第二个类型必须是结构体
	Info a("yede", 3.5);
	s.insert(a);
	a.name = "y2";
	a.score = 7.9;
	s.insert(a); 
	a.name = "y3";
	a.score = 8.9;
	s.insert(a); 
	set<Info>::iterator it;
	for (it = s.begin(); it != s.end(); it++) {
		cout << (*it).name<<":"<<(*it).score << " ";
	}
}
//输出y3:8.9 y2:7.9 yede:3.5
```
### 其他
#### 1. 集合求并集
需要使用`set_union`函数，`#incldue<algorithm>`
```C++
set<int> s1 = { 1,2,3,5,7 };
set<int>s2 = { 2,4,7,9,12,34,56,45 };
set<int>s3;
set_union(s1.begin(), s1.end(), s2.begin(), s2.end(), inserter(s3,s3.begin()));
for (auto it = s3.begin(); it != s3.end(); it++) {
	cout << *it << " ";
}
```

## multiset 多重集合容器
- `multiset`与`set`一样，也使用红黑树来组织元素数据的，唯一不同的是，`multiset`允许重复的元素键值插入，而`set`则不允许。
- 需要声明头文件`#include<set>`
### multiset插入
除了可以插入键值重复的元素，其余与set一致
```C++
multiset<string>a;
a.insert("nihao");
a.insert("1234");
a.insert("1234");
a.insert("12fe");
for (auto it = a.begin(); it != a.end(); it++) {
  cout << *it << " ";
}
//输出1234 1234 12fe nihao
```
### multiset删除
`erase()`删除与set略微不同，它可以删除`multiset`对象中的某个迭代器位置上的元素返回下一个元素的迭代器、某段迭代器区间中的元素返回下一个元素迭代器，也可以删除键值等于某个值的**所有重复元素同时返回删除元素的个数**。此外`clear()`可以清空元素 
```C++
multiset<string>a;
a.insert("nihao");
a.insert("1234");
a.insert("1234");
a.insert("12fe");
a.insert("1237");
int tmp = a.erase("1234");//删除所有"1234"元素，并返回元素总数2
cout << "delete " << tmp << " elements" << endl;

auto it=a.erase(a.begin());//删除第一个元素，返回下一个元素的迭代
cout << *it << endl;
for (auto it = a.begin(); it != a.end(); it++) {
  cout << *it << " ";
}
```
### 查找
`find()`与set中类似，若找到返回该元素的迭代器位置（若元素存在重复，返回第一个重复元素迭代器位置），未找到返回`end()`迭代器位置
```C++
multiset<string>a;
a.insert("nihao");
a.insert("1234");
a.insert("1234");
a.insert("1237");
a.insert("12fe");
auto it = a.find("1234");//找到"1234"返回第一个"1234"的位置
cout << *it << endl;
```

## map映照容器
- 元素数据由一个键值(key)和一个映照数据(value)组成的，键值与映照数据之间具有一一映照的关系。
- 采用红黑树来实现的，插入元素的键值**不允许重复**，比较函数只对元素的键值进行比较，元素的各项数据可通过键值检索出来。`map` 与 `set`采用的都是红黑树的数据结构，用法基本相似。
- 使用需要包括`#include<map>`
### 创建、插入和遍历
创建时key和value需要自己定义，比较规则缺省为key的由小到大顺序插入红黑树
```C++
// 构造函数初始化
std::map<std::string, int> m{{"CPU", 10}, {"GPU", 15}, {"RAM", 20}};

// 插入数值完成初始化
std::map<std::string, float> a;
a["yede"] = 8.9;
a["nide"] = 34.5;
a["hello"] = 4.6;

// 两种遍历方法
for (const auto& [key, value] : m)
	std::cout << '[' << key << "] = " << value << "; ";

for (auto it = a.begin(); it != a.end(); it++) {//这里使用auto也可以使用map<string, float>::iterator
  //(*it)的括号是不可省略的，否则相当于*(it.first)，原因考虑优先级
  cout << (*it).first << " : " << (*it).second << " ";
}
```
在上述代码中已包含了前向遍历的方法，map容器还可以通过反向迭代器，进行反向遍历map元素
```C++
map<string, float> a;
a["yede"] = 8.9;
a["nide"] = 34.5;
a["hello"] = 4.6;
map<string, float>::reverse_iterator it;
for (it = a.rbegin(); it != a.rend(); it++) {
  cout << (*it).first << " : " << (*it).second << " ";
}
```

### 删除
与set类似，`erase()`可以删除某个迭代器上的元素，等于某个键值的元素，一个迭代器区间上的所有元素。使用`clear()`清空
```C++
map<string, float> a;
a["yede"] = 8.9;
a["nide"] = 34.5;
a["hello"] = 4.6;
a.erase("nide");//删除键值为nide的元素
for (auto it = a.begin(); it != a.end(); it++) {
  cout << (*it).first << " : " << (*it).second << " ";
}
//输出为hello : 4.6 yede : 8.9
```
### 搜索
`find()`方法用于搜索某个键值，若找到返回键值所在迭代器的位置，否则返回`end()`位置，搜索效率极高
```C++
map<string, float> a;
a["yede"] = 8.9;
a["nide"] = 34.5;
a["hello"] = 4.6;
auto it = a.find("nide");
if (it != a.end())
  cout << "find" << endl;
else
  cout << "not found it " << endl;
```
### 自定义比较函数
与set类似，比较函数缺省时按照升序排列，有两种自定义比较规则的方式
1. 元素不是结构体，创建结构体，在结构体中重载()
```C++
struct myComp {
	bool operator()(const int& a, const int& b)const {
		if (a != b)
			return a > b; 
		else
			return a > b;
	}
};
int main(int argc, char* argv[]) {
	map<int, char, myComp> m; 
	m[25] = 'm';
	m[28] = 'k';
	m[10] = 'x';
	m[30] = 'a'; 
	map<int, char, myComp>::iterator it;
	for (it = m.begin(); it != m.end(); it++) {
		cout << (*it).first << " : " << (*it).second << endl;
	}
	return 0;
}
```
2. 元素是结构体，在结构体中实现比较函数
```C++
struct Info {
	string name;
	float score;
	bool operator<(const Info &a)const {
		return a.score < score;//按降序排列
	}
};
int main(int argc, char* argv[]) {
	map<Info,int> m; 
	Info in;
	in.score = 60;
	in.name = "jack";
	m[in] = 25;
	in.score = 70;
	in.name = "yede";
	m[in] = 40;
	in.score = 40;
	in.name = "nide";
	m[in] = 20;
	for (auto it = m.begin(); it != m.end(); it++) {
		cout << (*it).second << " : ";
		cout << ((*it).first).name << " " << ((*it).first).score << endl;
	}
	return 0;
}
//输出
// 40 : yede 70
// 25 : jack 60
// 20 : nide 40
```

## multimap 多重映照容器
- `multimap`允许插入重复键值的元素
- `multimap`的元素插入、删除、查找都与`map`不相同
- 使用需要`#include<map>`
### multimap创建，插入
```C++
multimap<string, double>m;
m.insert(pair<string, double>("nide", 500));
m.insert(pair<string, double>("hello", 600));
m.insert(pair<string, double>("wide", 450));
m.insert(pair<string, double>("wide", 450));//重复插入
for (auto it = m.begin(); it != m.end(); it++)
  cout << (*it).first << " : " << (*it).second << endl;
```
### 删除
与multiset类似，对于有重复的键值，删除操作会将其全部删除返回删除元素的数量；其他与map一直也可以删除某个迭代器位置上的元素，一个区间上的元素等
```C++
multimap<string, double>m;
m.insert(pair<string, double>("nide", 500));
m.insert(pair<string, double>("hello", 600));
m.insert(pair<string, double>("wide", 450));
m.insert(pair<string, double>("wide", 450));//重复插入
int a=m.erase("wide");//删除所有键值为wide的元素（哪怕value不一样）返回2
cout << a << endl;
for (auto it = m.begin(); it != m.end(); it++)
  cout << (*it).first << " : " << (*it).second << endl;
```
### 查找
`find()`返回重复键值中的第一个元素的迭代器位置，如果没有找到该键值，则返回end()迭代器位置
```C++
multimap<string, double>m;
m.insert(pair<string, double>("nide", 500));
m.insert(pair<string, double>("hello", 600));
m.insert(pair<string, double>("wide", 500));
m.insert(pair<string, double>("wide", 450));//重复插入
auto f = m.find("wide");
```

## deque使用
- 与`vector`类似采用线性表顺序存储结构，不同的是，`deque`采用分块的线性存储结构来存储数据
- 每块大小一般512字节，称为一个`deque`块，所有`deque`块使用一个`Map`块进行管理，每个`Map`数据项记录各个`deque`块首地址
- `deque`在首尾插入或删除的时间复杂度是常数
- 考虑到容器元素的内存分配策略和操作的性能时，`deque` 相对于 `vector` 更有优势
- 使用需要`#include<deque>`
### 创建deque对象
三种创建方式，与`vector`类似
```C++
deque<int> a;
deque<int> b(10);//创建10个int型元素deque对象b
deque<double> d(10,8.5);//创建10个double型元素的deque对象并初始化为8.5
```
### 插入元素
{% note warning %}
参考资料中指明从**头部和中间**插入元素，不会增加新元素，只将原有的元素覆盖；使用`push_back()`从尾部插入元素，会不断扩张队列。不理解
{% endnote %} 
使用`push_back()`方法从尾部插入元素，使用`push_front()`从头部插入元素，也可以使用`insert()`插入元素
```C++
deque<int> q;
q.push_back(1);
q.push_back(2);//从尾部插入两个元素
q.push_back(5);
q.push_front(10);
q.push_front(20); //从头部插入元素
q.insert(q.begin() + 4, 90);//在第5个元素前插入元素90
for (int i = 0; i < q.size(); i++)//以数组方式输出元素
	cout << q[i]<<" ";
```
### 遍历方式
1. 使用下标访问与数组相同，参考上段代码
2. 以前向迭代器方式遍历
```C++
deque<int> q;
q.push_back(1);
q.push_back(2);//从尾部插入两个元素
q.push_back(5);
q.push_front(10);
q.push_front(20); //从头部插入元素
q.insert(q.begin() + 4, 90);//在第5个元素前插入元素90
for (deque<int>::iterator i = q.begin(); i != q.end(); i++)
	cout << *i << " ";
```
3. 以反向迭代器方式遍历
```C++
for (deque<int>::reverse_iterator i = q.rbegin(); i != q.rend(); i++)
	cout << *i << " ";
```
### 删除元素
可以从队列的首部，尾部，中部删除元素，并可以清空容器，
1. 使用`pop_front()`元素从头部删除元素
```C++
deque<int> q;
q.push_back(1);
q.push_back(2);
q.push_back(5);
q.push_front(10);
q.push_front(20); 
q.pop_front();//从头部删除元素
for (auto i = q.begin(); i != q.end(); i++)
	cout << *i << " ";
//输出10 1 2 5
```
2. 使用`pop_back()`方法从尾部删除元素
```C++
deque<int> q;
q.push_back(1);
q.push_back(2);
q.push_back(5);
q.push_front(10);
q.push_front(20); 
q.pop_back();//从尾部删除元素
for (auto i = q.begin(); i != q.end(); i++)
	cout << *i << " ";
// 输出20 10 1 2
```
3. 使用`erase()`方法从中间删除元素
```C++
q.push_back(1);
q.push_back(2);
q.push_back(5);
q.push_front(10);
q.push_front(20); 
q.erase(q.begin() + 3);//删除第4个元素2
for (auto i = q.begin(); i != q.end(); i++)
	cout << *i << " ";
//输出 20 10 1 5
```
4. 使用`clear()`方法清空`deque`对象

## list双向链表容器
- `list`数据结构为双向循环链表，每个节点有前驱指针，数据，后继指针
- 对链表的任意位置元素进行插入，删除和查找都很快
- list对象节点不要求在一段连续的内存中，所以对于迭代器只能使用`++`或者`--`进行移动
- 使用需要`#incldue<list>`
### 创建list对象
与`vector`容器类似
```C++
list<int> l;//创建空链表
list<int> l(10);//创建一个具有10个元素的链表
```
### 插入与遍历
有三种方法进行插入，三种方法插入后链表自动扩张
1. 使用`push_back()`从尾部插入元素
2. `push_front()`从首部插入元素
3. `insert()`在迭代器位置插入元素
```C++
list<int> q;
q.push_back(1);
cout << q.size() << endl;//输出链表大小方便与后续对比
q.push_back(5);
cout << q.size() << endl;
q.push_front(10);
q.push_front(20);
q.insert(++q.begin(),90);
for (auto i = q.begin(); i != q.end(); i++)
	cout << *i << " ";
// 输出
// 1
// 2
// 20 90 10 1 5
```
遍历可以使用前向迭代器对链表遍历，也可以使用反向迭代器遍历与之前的大部分遍历相同，不再赘述
### 元素删除
1. 使用`remove()`方法删除链表中的一个元素，值相同的元素都会被删除
2. 使用`pop_back()`删除链尾元素，或使用`pop_front`删除链首元素
3. 使用`erase()`删除迭代器位置上的元素
4. 使用`clear()`方法清空链表
```C++
list<int> q;
//先构建链表
q.push_back(1);
q.push_back(5);
q.push_back(5);
q.push_back(34);
q.push_front(10);
q.push_front(20);
q.insert(++q.begin(),90);

q.pop_back();//删除尾部元素34
q.pop_front();//删除首部元素20
q.remove(5);//删除所有为5的元素
for (auto i = q.begin(); i != q.end(); i++)
	cout << *i << " ";
q.clear();//清空链表
```
### 元素查找
使用`find()`查找算法（不是链表的成员），查到返回迭代器位置，否则返回`end()`迭代器，使用需要`#incldue<algorithm>`
```C++
list<int> q;
//先构建链表
q.push_back(1);
q.push_back(5);
q.push_back(34);
q.push_front(10);
auto it = find(q.begin(), q.end(), 5);
if (it != q.end())
	cout << "find"<<endl;
// 输出find
```
### 元素排序
使用`sort()`方法（不是`algorithm`中的`sort()`）对链表元素进行升序排列
```C++
q.push_back(1);
q.push_back(16);
q.push_back(34);
q.push_front(10);
q.sort();
for (auto i = q.begin(); i != q.end(); i++)
	cout << *i << " ";
// 输出1 10 16 34
```
### 删除连续重复元素
`unique()`方法可以删除**连续重复**(与重复含义不同)的元素只保留一个
```C++
q.push_back(1);
q.push_back(16);
q.push_back(9);
q.push_back(16);
q.push_back(16);
q.push_back(34);
q.push_front(10);
//执行删除重复之前为10 1 16 9 16 16 34
q.unique();//删除之后为10 1 16 9 16 34
for (auto i = q.begin(); i != q.end(); i++)
	cout << *i << " ";
```

## bitset位集合容器
[参考资料](https://en.cppreference.com/w/cpp/utility/bitset)
- 每个元素只占一个bit位，取值为0或1
- 使用需要`#include<bitset>`
- 第0位是最低位，第n位是最高位

| 方 法          | 功能                               |
|--------------|----------------------------------|
| b.any()      | b 中是否存在置为 1 的二进制位                |
| b.none()     | b 中不存在置为 1 的二进制位                 |
| b.count()    | b 中置为 1 的二进制位的个数                 |
| b.size()     | b 中二进制位的个数                       |
| b.test(pos)  | b 中在 pos 处的二进制位是否为 1             |
| b.set()      | 把 b 中所有二进制位都置为 1                 |
| b..set(pos)  | 把 b 中在 pos 处的二进制位置为 1            |
| b.reset()    | 把 b 中所有二进制位都置为 0                 |
| b.reset(pos) | 把 b 中在 pos 处的二进制位置为 0            |
| b.flip()     | 把 b 中所有二进制位逐位取反                  |
| b.flip(pos)  | 把 b 中在 pos 处的二进制位取反              |
| b.to_ulong() | 用 b 中同样的二进制位返回一个 unsigned long 值 |
| os << b      | 把 b 中的位集输出到 os 流                 |
### 创建bitset对象
创建bitset对象时，必须指定容器大小，且大小一旦指定就不能修改
```C++
bitset<1000> b;
cout << b;
//输出1000个0
```
### 为元素赋值
1. 使用下标`b[0]=1;`
2. 使用`set()`方法`b.set();`将所有元素设置为1
3. 使用`set(pos)`方法`b.set(1,1);`将第2个元素设置为1
4. 使用`reset(pos)`方法`b.reset(3)`将第4个元素设置为0
### 元素输出
1. 使用下标输出，与数组类似
2. 向输出流输出全部元素
```C++
bitset<10> b;
cout << b;
// 输出0000000000
```


## stack堆栈容器
- 后进先出（LIFO）线性表，插入和删除元素都只在表的一端进行。
- 使用需要`#include<stack>`
- 只提供入栈、出栈、栈顶元素访问和判断是否为空等几种方法。
	- `push()`方法将元素入栈；
	- `pop()`方法出栈；
	- `top()`方法访问栈顶元素；
	- `empty()`方法判断堆栈是否为空，为空的，返回1，否则返回0。
	- `size()`方法返回当前堆栈中有几个元素。

## queue队列容器
- 先进先出（FIFO）线性存储表，插入只能在队尾，删除只能在队首
- 使用需要`#include<queue>`
- 成员函数与栈（stack）类似
	- `push()`入队
	- `pop()`出队
	- `front()`读取队首元素
	- `back()`读取队尾元素
	- `empty()`队列是否为空
	- `size()`获取元素数目
```C++
queue<string> q;
q.push("first");//队尾插入元素
q.push("second");
cout<<q.front()<<endl;//输出队首元素first
q.pop();//队首出队
cout<<q.front()<<endl;//输出second
```

## priority_queue优先队列容器
- 与`queue`一样，只能队尾插入元素，队首删除，不同在于队列中最大的元素总是位于队首。表现上相当于给队列中的元素由大到小排序
- 元素比较规则默认为由大到小，可以自定义比较规则
- 使用需要`#include<queue>`，使用方式除了比较规则外与`queue`一致
### 自定义比较规则
1. 元素类型为结构体，重载`<`修改队列优先性（也可以按照2的方法）
```C++
#include<queue>
#include <iostream>
#include<string>
using namespace std; 
struct Info {
	string name;
	int va;
	bool operator<(const Info& a)const {
		return a.va < va;///按va由小到大排列。如果要由大到小排列，使用“>”号即可
	}
};
int main(int argc, char* argv[]) {
	priority_queue<Info>q;
	Info a;
	a.name = "nihao";
	a.va = 40;
	q.push(a);
	a.name="yede";
	a.va = 30;
	q.push(a);
	a.name = "tade";
	a.va = 50;
	q.push(a);
	while (!q.empty()){
		cout << q.top().name << " : " << q.top().va << endl;
		q.pop();
	}
}
// 输出为
// yede : 30
// nihao : 40
// tade : 50
```
2. 元素不是结构体类型，重载`()`定义优先级
```C++
#include<queue>
#include <iostream>
#include<string>
using namespace std; 
struct com{
	//参数类型需要与实例化的类型一致
	bool operator()(const int& a, const int& b)const {
		return a > b;////由小到大排列采用“>”号；如果要由大到小排列，则采用“<”号
	}
};
int main(int argc, char* argv[]) {
	//定义优先队列，元素类型为 Info 结构体，显式说明内部结构是 vector
	priority_queue<int, vector<int>, com>q;
	q.push(3);
	q.push(40);
	q.push(20);
	q.push(24);
	while (!q.empty()) {
		cout << q.top()<< " ";
		q.pop();
	}

}
// 输出为3 20 24 40
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

## 参考资料
[^1]:[C++STL](http://c.biancheng.net/view/6675.html)
[^2]:[C++STL标准手册](https://cplusplus.com/reference/stl/)
[^3]:[cppreference](https://en.cppreference.com/w/)