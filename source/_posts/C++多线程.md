---
title: C++多线程
date: 2023/2/19 14:14:12
categories: learning-notes
excerpt: 总结C++多线程的使用
hide: true
tag: 
- markdown
- C++
---

使用线程需要`include<thread>`
## 线程启动
构造thread对象，可填以下参数：
1. 使用lambda表达式
2. 使用函数对象，需要注意传递形式，直接传入`background_task()`将出现错误（传递临时变量将被认为是函数声明）
3. 有函数调用符类型的实例
4. 函数
```C++
#include<iostream>
#include<thread>
using namespace std;
class background_task
{
public:
    void operator()() const
    {
        cout << "nihao" << endl;
        cout << "hello" << endl;
    }
};
void do_some_work(){
  cout<<"do_some_work"<<endl;
}
int main() {
  background_task f;
  // 下述四种选择一种即可
  // lambda表达式
  thread my_thread([] {
      cout << "nihao" << endl;
      cout << "hello2" << endl;
  });
  // 函数
  void do_some_work();
  thread my_thread(do_some_work);
  // 有函数调用符类型的实例
  background_task f;
  thread my_thread(f);
  // 函数对象
  thread my_thread((background_task()));  
  thread my_thread{background_task()};   

  my_thread.join();
  return 0;
}
```
## 等待线程
使用`join()`方法，清理线程相关的存储部分，`thread`对象将不再与已经完成的线程有任何关联。故只能对一个线程使用一次`join()`，可使用`joinable()`判断某个线程是否使用过`join()`

## 参数传递
1. 传递引用。在线程中直接传递引用，`thread`的构造函数会无视函数期待的参数类型，并盲目的拷贝已提供的变量，传递给函数的参数是`data`变量内部拷贝的引用，而非数据本身
```C++
#include<iostream>
#include<thread>
using namespace std;
typedef int widget_id;
typedef int widget_data;
void update_data_for_widget(widget_id w, widget_data& data) {
	data = 4;
}
void oops_again(widget_id w)
{
	widget_data data=10;
  // 使用ref将参数转换成引用的形式，从而可将线程的调用改为以下形式：
  // thread t(update_data_for_widget, w, ref(data));
	thread t(update_data_for_widget, w, data);//直接使用该语句将会报错
	cout << " current data: " << data << endl;;
	t.join();
	cout << " after thread data: " << data << endl;
}
int main() {
	oops_again(5);
}

```
## 其他
获取线程id和CPU最多的核芯
```C++
	cout << "hardware_concurrency: " << thread::hardware_concurrency() << endl;
	cout <<"current thread_id: " << this_thread::get_id() << endl;
```
## 互斥量
需要`#include<metux>`,通过实例化metux对象创建互斥量，调用成员函数lock()和unlock()进行上锁和解锁（注意每个函数出口都要unlock()包括异常退出）；也可以使用模板类lock_guard，在构造函数中提供已锁的变量在析构时解锁
metux的示例程序，在两个函数中对数据some_list的访问是互斥的，当函数返回的是保护数据的指针或引用时，会破坏对数据的保护，因此切勿将受保护数据的指针或引用传递到互斥锁作用域之外
```C++
#include <list>
#include <mutex>
#include <algorithm>
#include<iostream>
std::list<int> some_list;
std::mutex some_mutex;   
void add_to_list(int new_value)
{
	std::cout << "current thread: " << std::this_thread::get_id() << std::endl;
	std::lock_guard<std::mutex> guard(some_mutex);
	some_list.push_back(new_value);
	for (auto ele : some_list) {
		std::cout << ele << " ";
	}
	std::cout << std::endl;
}
bool list_contains(int value_to_find)
{
	std::cout << "current thread: " << std::this_thread::get_id() << std::endl;
	std::lock_guard<std::mutex> guard(some_mutex); 
	bool tmp = std::find(some_list.begin(), some_list.end(), value_to_find) != some_list.end();
	std::cout << tmp;
	return tmp;
}

int main() {
	std::thread t1(add_to_list,13);
	std::thread t2(add_to_list, 114);
	std::thread t3(list_contains, 13);
	t1.join();
	t2.join();
	t3.join();
	
	return 0;
}
```







[^1]:[C++ Concurrency In Action](https://www.bookstack.cn/read/Cpp_Concurrency_In_Action/content-chapter2-2.1-chinese.md)