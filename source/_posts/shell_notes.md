---
title: shell学习笔记
date: 2022/8/23 14:14:12
updated: 2022/9/23 15:34:00
categories: learning-notes
excerpt: 记录shell学习笔记
hide: false
tag: markdown
---

## md部分语法说明
> 这里是引用

---

`using namespace std;//这里是代码`

---

|clo1|clo2|
|---|---|
|1|2|

---

## 常用指令
- `ps -f`查看当前所有进程

## shell注意事项
- 重定向
    标准输入文件，stdin文件描述符0
    标准输出文件，stdout文件描述符1
    标准错误文件，stderr文件描述符2
    重定向到/dev/null文件=禁止输出（任何内容均丢弃）
    ```shell
    # 将hello.txt文件中的内容传给cat ,然后再传给hello2.txt（覆盖原内容）
    cat < hello.txt > hello2.txt

    # 将hello.txt文件中的内容传给cat ,再追加给hello2.txt（不覆盖原内容）
    cat < hello.txt >> hello2.txt

    # 将错误信息重定向到文件
    $ command 2>file
    # 将错误信息和标准输出信息重定向到文件
    $ command > file 2>&1
    ```

- `let`指令无法使用，将开头的`#!/bin/sh`换为`#!/bin/bash`
语法
- 变量名和等号之间不能有空格
- 0代表true，1代表false

## 执行脚本

以下两种启动子shell执行脚本，执行完成后回到父shell，子shell可以使用父shell的变量
绝对/相对路径（需要x权限）
```shell
./hello.sh
绝对路径/hello.sh
```
使用bash执行（不需要x权限）
```shell
sh hello.sh
```

使用source执行，不启动子shell
```shell
. hello.sh
source hello.sh
# source是shell中内嵌的，source中不会启动子shell
```
![](/img/note-picture/4.png)

## 权限说明
![](/img/note-picture/3.png)


| 符号  |  d | -  | l  | r  | w | x |
|---|---|---|---|---|---|---|
| 含义  | 文件夹  | 文件  | 链接  | 可读  |可写   | 可执行  |

- 第一个字符是类型，后每三个为一组，分别表示文件的**拥有者**，文件的**所属组**，文件的**其他用户**所拥有的权限
- 第一个yede表示拥有者，第二个为这个拥有者所在组

赋于执行权限
```shell
chmod +x ./filename # 使脚本具有执行权限
./filename # 执行脚步
```

## 变量说明
- `export variable_name`将局部变量提升为全局变量（不需要$），子bash中的修改只对子bash有效
- `set`包含所有系统和用户定义的全局变量、局部变量；
    `env`中是系统中的全局变量；
    局部变量在子shell中无效
- `readonly variable_name`设置变量只读
    ```shell
    myUrl="https://www.google.com"
    readonly myUrl
    ```
- `unset variable_name`删除变量，测试之后需要及时删除以免占用内存,`readonly`无法`unset`
    ```shell
    unset variable_name
    ```
- 特殊变量
    |参数|含义|
    |---|---|
    |$#|参数个数|
    |$*|所有参数，以"$1 $2 … $n"的形式输出所有参数|
    |$@|所有参数，以"$1","$2" … "$n" 的形式输出所有参数|
    |$$|脚本运行的当前进程ID号|
    |$!	|后台运行的最后一个进程的ID号|
    |$?|最后命令的退出状态。0没有错误，其他值有错|
- 字符串
    单引号内任何字符都会原样输出，转义符无效；双引号里可以有变量，有转义字符;也可以不要引号
- 数组
    定义方式array_name=(value0 value1 value2 value3)，用空格分开
    ```shell
    # 读取某一元素的值
    valuen=${array_name[n]}
    #  @可以获取数组中的所有元素
    echo ${array_name[@]}
    # 取得数组元素的个数
    length=${#array_name[@]}
    length=${#array_name[*]}
    # 取得数组单个元素的长度
    lengthn=${#array_name[n]}
    ```

## 运算
- 使用`expr`完成计算
- 使用`$[]`或者`$(())`
- 关系运算符,使用如右`[ $a -eq $b ]`
    |运算符|说明|
    |---|---|
    |-eq | equal 相等|
    |-ne | not equal 不相等|
    |-lt | less than 小于|
    |-le | less equal 小于等于|
    |-gt | greater than 大于|
    |-ge | greater equal 大于等于|
    |! | 非|
    |-r | or 或|
    |-a | and 与|
- 逻辑运算符&& ,||可借此形成三元表达式
- 字符串运算符，文件测试运算符，不常用,见[链接](https://www.runoob.com/linux/linux-shell-basic-operators.html)

示例
```shell
# expression
a=`expr 1 + 2`
echo $a

expr 4 \* 2 # 乘法运算
echo $[ 1+2 ]
echo $((3*5))

echo $[ 3 == 3 ]

# 在比较运算中使用[]不能用<>，但(())中可以使用<>（测试中好像都行）
# 进行加一操作的两种方法
let a++
let a+=1
a=$[$a+1]

```
注意：
- 在`expr`表达式和运算符之间要有空格
- 条件表达式注意空格`[ 2 == 3 ]`（测试中主要是[]两侧的空格）
- 表达式中的=可用于检测字符串是否相等，==检测

## if,for,while,case,until语句
**if**
```shell
# if-slse
if condition
then
    command1 
    command2
    ...
    commandN
else
    command
fi

# if-elif-else
if condition1
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi
```

**for**
```shell
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done

for ((初始值;循环控制条件;变量变化))
do
    command
done

# 实例
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```

**while**
```shell
while condition
do
    command
done
```

**多选择语句**
```shell
# ;;相当于break
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2)
    command1
    command2
    ...
    commandN
    ;;
esac
```

## 读取输入read
-t 指定等待时间，不加则一直等待
-p 指定提示语
```shell
# 实例，name表示输入数据名称
read -t 10 -p "please input your name: " name
echo $name
```

## 函数
系统函数
basename获取当前文件名
dirname获取路径
```shell
# 函数的格式，[]内的是可省略部分
[ function ] funname [()]
{

    action;

    [return int;]

}
```
注意：
return后的数值n(0-255)

## 字符串处理
**字符串截取**
```shell
从start开始，截取长度为length的字符串
格式：${str:start:length}  
echo ${str:5:8}
```

## 指令
- wc 统计文件信息
    |符号|含义|
    |---|---|
    |-l|统计行数|
    |-m|统计字符数|
    |-w|统计字数|
    |-c|统计字节数|

    ```shell
    # 统计3个文件的信息，输出分别表示行数、单词数、字节数
    wc filename1 filename2 filename3
    3 94 234 filename1
    ...
    ```
- uptime 获取服务器信息
    ```shell
    uptime
    08:21:34 up 36 min,  2 users,  load average: 0.00, 0.00, 0.00
    
    #当前服务器时间：    08:21:34
    #当前服务器运行时长  36 min
    #当前用户数          2 users
    #当前的负载均衡      load average  0.00, 0.00, 0.00，分别取1min,5min,15min的均值
    ```
- date 有关日期的指令

- head 
    ```shell
    # 查看第一行内容
    head -1 filename
    ```
- tail
    与head类似
    ```shell
    # 查看最后一行内容
    tail -1 filename
    ```
- ssh 远程连接
    ```shell
    # 连接远程服务器并执行以下指令
    ssh user@公网ip "cd /home ; ./a.sh"
    ```
