---
title: C++STL泛型进阶
date: 2023/2/19 14:14:12
categories: learning-notes
excerpt: 侯捷STL源码剖析笔记
hide: false
tag: 
- markdown
- C++
- C
---
C++标准库>C++标准模板库
标准库以head files形式存在
C++标准库的header files不带后缀，如#include<vector>
新式C`header files`不带.h后缀，如#include<cstdio>
旧式的`header files`带.h后缀，仍可用，#include<stdio.h>
新式`headers`的所有组件封装在`namespace std`中
旧式headers内组件不会封装在namespace std中
## C++STL六大部件
容器Containers
分配器Allocators
算法Algorithms，函数模板
迭代器Iterators
适配器Adapters
仿函数Functors
迭代器可以看做是一种泛化的指针
`vector<int,allocator<int>>`分配器有缺省值，一般可以省略
```C++
#include <iostream>
#include<vector>
#include<algorithm>
#include<functional>
using namespace std;	

int main() {
	vector<int, allocator<int>>a = { 25,124,566,112,34,56,2,12,344,32 };
	cout << count_if(a.begin(), a.end(), 
		not1(bind2nd(less<int>(), 40)));//判断>=40的数据个数
	//less是functor
}
```
begin()指向第一个元素，end()指向最后一个元素的下一个位置
C++11后有`range-based for statement`简化遍历，以及智能指针auto
```C++
// 
// for(decl: coll){
//   statement
// }

vector<int, allocator<int>>a = { 25,124,566,112,34,56,2,12,344,32 };
for (int i : {1, 2, 3, 5, 3, 2, 7, 12}) {
  cout << i << " ";
}
cout << endl;
for (auto it : a) {
  cout << it << " ";
}
//输出1 2 3 5 3 2 7 12
//25 124 566 112 34 56 2 12 344 32
```

结构分类：序列容器，关联容器，无序容器(Unordered Containers，顺序不确定，由hashtable实现)
C++11新增array,forward-list,unordered containers，
1. array一段固定的空间，无法扩充，vector一端可扩充，均为连续空间，deque两端可扩充，分段连续
2. list双向链表，forward-list单向链表，均为非连续空间
3. set，map,multimap,multiset使用红黑树实现
4. unordered map,unordered map实际是hash table中的separate chain

1. sequence containers
	- `vector`空间不够，扩容为原来的2倍，将原来的元素搬到新的`vector`中，可以通过`capacity()`查看分配的空间
	- `list`有自己的排序函数，每次扩充一个节点
	- `forward_list`没有`back()`,`size()`，有自己的排序函数`sort()`
	- `deque`分段连续，一段称为`buffer`
	- `stack`与`queue`内部使用`deque`实现，有的地方认为`stack`与`queue`是`Adapter`，分别具有先进后出，先进先出的特点
	- `slist`也是单向列表，但不是STL内容，在`ext/slist`头文件中
2. Associative Containers
	- `multiset`自己有find函数，查找较快，红黑树结构
	- `multimap`插入不能使用`[]`，只能通过`insert()`插入，红黑树结构
3. unordered containers
	- `unordered_multiset`通过`hash`分配给各个`bucket`，每个`bucket`有一个链表放元素，可通过`bucket_count`查询bucket个数，bucket数量多于元素数量，自己又查找方法
	- `unordered_multimap`与`unordered_multiset`类似，使用`hash table`做底层


## 基础知识
Dev-C++的STL源码路径`...\Dev-Cpp\MinGW64\lib\gcc\x86_64-w64-mingw32\4.9.2\include\c++\bits`
### OOP与GP
OOP(object-oriented programming)面向对象，数据与方法放在一起，如`list`中的`sort()`方法
GP（generic programming）泛型，数据与方法分离，将容器与算法（对应数据与方法）分开，两者通过iterator沟通
### 运算符重载
[参考资料](https://en.cppreference.com/w/cpp/language/operators)
包含哪些操作符不能重载，能重载的操作符如何重载
### 模板
类和函数都有模板，在声明时，类需要`<>`指明数据类型，但函数不用`<>`指明数据类型
```C++
// 下面两种方式均可
template<class T>
template<typename T>
```
### 泛化与特化
泛化，特化，偏特化，例子如下，优先匹配特化类型，没有则采用泛化类型
```C++
// 泛化
template<class T>
struct __type {
	typedef int miao;
	typedef double dmiao;
};

// 特化，也被称为全特化
template<>
struct __type<int>
{
	typedef int miao;
};

// 偏特化,partial specialization
// 第一种偏特化
template<class T>
struct __type {
	typedef int miao;
	typedef double dmiao;
};

template<class T>
struct __type<T*>
{
	typedef int miao;
};

template<class T>
struct __type<const T*>
{
	typedef int miao;
};
// 第二种偏特化，对一个typename取具体的数据类型
template<class T,class T2>
struct __type {
	typedef int miao;
	typedef double dmiao;
};

template<class T2>
struct __type<int,T2>
{
	typedef int miao;
};
```
## 分配器
GNU4.9中，在`__gnu_cxx`命名空间，需要`#include<ext\array_allocator>`，有以下分配器
1. array_allocator
2. mt_allocator
3. debug_allocator
4. pool_allocator
5. bitmap_allocator
6. malloc_allocator
7. new_allocator
```C++
//不会直接使用分配器
allocator<int>a;
int* p = a.allocate(1);//分配一个单元
a.deallocate(p, 1);//还回一个单元和一个指针
```
各个版本中的分配器
1. VC6 STL对allocator的使用，没有特殊设计
	- allocate时，调用operator new，最底层是使用malloc（在使用malloc申请空间时，多个小字节元素申请的空间overhead会很大，每一块空间需要记录空间大小，因此有较多额外开销）
	- deallocate，调用operator delete，最底层调用free
	```C++
	// ()临时对象
	int* p = allocator<int>().allocate(512, (int*)0);//给出0或1等都无所谓
	// 在归还内存时还需要告知申请时的空间
	allocator<int>().deallocate(p, 512);
	```
2. BC5中的allocate和deallocate与VC相同，无特殊设计
3. GNU2.9中allocator的allocate与deallocate与VC相同，无特殊设计，但默认使用的是alloc分配器。GNU2.9中alloc有独特的设计，在`<stl_alloc.h>`中alloc通过16个类似链表的东西实现内存分配，每一块增加8，能够申请一大块空间，避免额外开销来记录每一个空间的大小
4. G4.9，默认使用allocator，`<bits/allocator.h>`中的allocator类继承new_allocator类(在`<bits/new_allocator.h>`中)，但new_allocator中的allocate和deallocate方法与VC一致，在G4.9中有很多扩展allocator，其中的`pool_alloc`就是G2.9的`alloc`

## 迭代器与traits
- 迭代器需要有5种`associated types`（迭代器作为算法和容器的桥梁，在使用算法时，迭代器需要给算法提供这5个东西），能够得出迭代器`category`（种类），`difference`（迭代器距离），`value`（被指元素数据类型），`reference` 和`pointer`（后两种在标准库中未被采用）
- 注意：`difference_type`大小超过其数据类型（定义中是`ptrdiff_t`，实际是unsigned int）所允许的大小时可能会失效
- 有两种迭代器`class iterators` 和`non-class iterators`（即pointer，退化的迭代器），前者可以方便的获取5种`associated types`，后者则不行，因此需要`iterator traits`萃取机，分离两种迭代器，`traits`如果接收`pointer`将使用偏特化
```C++
template<class T>
struct iterator_traits<T*>
{
	typedef random_access_iterator_tag iterator_category;
	typedef T						   value_type;
	typedef ptrdiff_t				   difference_type;
	typedef T*						   pointer;
	typedef T&						   reference;
};

template<class T>
struct iterator_traits<const T*>
{
	typedef random_access_iterator_tag iterator_category;
	typedef T						   value_type;
	typedef ptrdiff_t				   difference_type;
	typedef const T* pointer;
	typedef const T& reference;
};
```
### 迭代器种类
五种迭代器category，种类使用对象表现，下一行继承上一行的迭代器（通过类提高算法复用，如实现input_iterator对应的算法，其子类若无特殊性质也可以使用该算法）
input_iterator_tag/ouput_iterator_tag
forward_iterator_tag
bidirectional_iterator_tag
random_access_iterator_tag
```C++
//需要包含#include<typeinfo>
vector<int>::iterator it;
// 得到指针的type
cout << typeid(it).name() << endl;
cout << typeid(istream_iterator<int>()).name() << endl;
cout << typeid(ostream_iterator<int>(cout, "")).name() << endl;
```

## 容器
缩进表示后者由前者衍生，复合关系，即后者中有一个前者，通过`sizeof()`函数可以查看容器的大小（与容器的`size()`方法不同，是查看容器中的指针等结构所占大小），如`cout << sizeof(vector<int>) << endl;`
```
<!-- sequence -->
array
vector
	heap
		priority_queue
list
slist，非标准
deque
	stack
	queue

<!-- Associative Containers -->
rb_tree，非公开
	set
	map
	multiset
	multimap
<!-- unordered containers -->
hashtable，非公开
	hash_set，非标准
	hash_map，非标准
	hash_multimap，非标准
	hash_multiset，非标准
```
C++11中slist改名为forward_list，hash_set,hash_map改名为unordered_set等，并新加array

### list
- list实际是一个双向循环链表，并且具有额外的一个元素，即最后一个元素的下一个元素。
- iterator是list中的一个类，这个类对指针常用的操作符有重载
- 版本区别：
	- 在GNU2.9中列表的指针使用`void*`类型，迭代器需要三个参数`<T,T&,T*>`，链表类与相关的类关系不复杂，迭代器定义中含有5中associated types；
	- 在GNU4.9版本中迭代器只有一个参数`<T>`在迭代器内实现`T*`和`T&`，并且链表指针成为指向自己的类型，即`_list_node_base`，链表类的相关类增加出现继承，组合等多种关系，迭代器定义中含有5中associated types。
- iterator++返回不带reference所以不能++两次，与`int i=0;i++;`同理，但++iterator返回reference，可以前++两次

### vector
1. vector扩充。当vector空间不够时会在内存中找到另一块**2倍大**的空间（如果找不到则生命结束），然后将原来的元素拷贝过来，并将原来的元素删除，因此在使用时需要注意扩容问题，vector中的各个指针如下图所示
![](../img/C%2B%2BSTL%E6%B3%9B%E5%9E%8B%E8%BF%9B%E9%98%B6/vector.png)
2. 对于空间是连续的容器往往会提供`[]`操作符
3. 版本差别：
	- G2.9中使用一个指针做迭代器（退化的迭代器），因此在使用迭代器时先输入萃取机`traits`再获取5个`associated type`
	- G4.9增加了继承关系
vector在扩容时的优化：
1. 使用`emplace_back()`代替`push_back()`，尽量避免不必要的对象拷贝操作。`emplace_back()`在容器末尾直接构造元素，而`push_back()`需要先构造一个对象，再将对象拷贝到容器中。
2. 避免频繁插入或删除元素，这会导致`vector`容器的动态扩容和缩容，影响性能。
3. 可以先将元素保存到一个临时`vector`中，再使用`swap()`函数一次性将所有元素插入到目标`vector`中。
### array
array是不能扩充的，一经创建大小就不能改变，没有构造函数和析构函数，只有pointer，退化的迭代器，4.9版本也多了很多继承关系
### deque
- 分段串联起来，使用一个类似vector的结构，每个位置存储指向对应buffer的指针，表现上看是（分段）连续的，可以使用`[]`访问元素
- deque的迭代器是一个类，有含有first(buffer开始位置),cur（当前位置）,last（buffer结束位置，与first共同确定buffer的边界）,node(在控制中心vector的位置，在一个buffer走到尾时会使用)，含有三个迭代器，start，finish和iterator
- deque的插入函数`insert()`会根据被插入元素位置前后的元素个数决定，把其余数据向头端推动或者向后端推动
- 为了模拟连续空间。对大部分操作符进行了重载。
- 版本区别。G2.9版本deque是单一的一个类，可以指派buffer_size，G4.9变为多个，且不能指定buffer_size，512/元素大小作为buffer大小
- 扩容。deque的控制中心使用与vector类似的结构，因此当控制中心满后也需要扩容，与vector扩容类似，但deque在将原来元素复制到新空间时，会复制到新空间的中间
### stack，queue
- 底层默认使用deque实现，只需要调用deque的一些操作即可
- 不提供迭代器，不可遍历
- 模板类的第二个参数是`class Sequence=deque<T>`即默认底层使用deque，也可使用list做底层实现，但deque效率更高；都不可选择map或set作为底层
- 区别：queue不可使用vector做底层（局部可以），但stack可以使用vector做底层
### rb-tree（非公开）
- 红黑树采用中序遍历，有两种insertion操作，`insert_unique()`插入唯一值和`insert_equal()`插入值可以重复
- 红黑树的header节点是一个空的仅有辅助功能的节点
- 迭代器可以修改元素值
- 红黑树模板有5个参数，key，value(key+data)，keyOfValue（如何取到key），compare（比较规则），Alloc(分配器)
### set/multiset
- 以红黑树为底层，value与key一致，value就是key，不能使用迭代器修改元素值（因为key就是value），通过const_iterator实现
- set的insert使用红黑树的`insert_unique()`；multiset的insert使用insert_equal()
- set所有操作依靠红黑树的方法实现，因此也可以将set看做一个container Adapter
### map/multimap
- 以红黑树为底层，要求放入四个模板参数（`class Key,class T,class Compare=less<Key>,class Alloc=alloc`）后两个参数有默认值，因此一般只需要给出两个参数即可，调用的底层的红黑树参数`Key, pair<const Key,T>, select1st<pair<const Key,T>>, Compare, Alloc`，在VC6中没有`select1st()`
禁止修改key而可以修改data，通过设置key为const类型实现
map可以通过`[]`修改或插入值，但multimap不能。`[]`返回key所对应的data，如果key不存在则新建一个以key，data=默认值的节点并被返回，`[]`内部先`lower_bound()`再调用`insert()`，因此`insert()`效率更高
### hashtable(非公开)
- 同一位置出现多个元素使用单向链表（VC使用双向链表），即`separate chaining`
- hashtable（实际使用vector）中的每一个位置称为一个bucket，一般bucket数量会选择素数，当一个bucket所挂的链表过长时（长度超过总bucket数量），将hashtable进行扩充（扩充为原来2倍附近的素数）以便将元素打散重排，没扩充一次都会将所有元素重新排列
- hashtable要求给出六个模板参数`<class Value,class Key,class HashFcn, class ExtractKey, class EqualKey, class Alloc=alloc>`，HashFcn仿函数，用于计算编号放入对应的bucket；ExtractKey如何取出key；EqualKey指定什么叫key相等。
- hash<>具有多个特化版本，用于产生编号以便排序
- 迭代器根据链表是否为单向或双向，使用bidirectional和forward
### unordered_set,unordered_multiset,unordered_map,unordered_multimap
- bucket_size(i)查询第i个bucket含有的元素个数
- bucket数量一定大于元素个数

## 算法
算法首先要根据迭代器获取必需的信息（5种associated type）
iterator traits与type traits对算法效率有较大影响，算法需要根据数据特性选择最高效的算法
accumulate
## 参考资料
[^1]:[侯捷STL源码剖析](https://www.youtube.com/watch?v=_dzIvOmXfKI&list=PLTcwR9j5y6W2Bf4S-qi0HBQlHXQVFoJrP&index=19)