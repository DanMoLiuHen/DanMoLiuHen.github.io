---
title: django学习笔记
date: 2023/1/29 16:15
categories: learning-notes
excerpt: django学习笔记
hide: false
tag: markdown
---

## 创建项目
`django-admin startproject <pro-name>`

默认项目结构说明，创建一个first项目
```
first
  manage.py             项目管理，启动项目，创建app，数据管理等
  first
    __init__.py
    settings.py         项目的配置文件
    urls.py             url和函数的对应关系
    wsgi.py             接收网络请求
    asgi.py             asynchronization，同上，可接受异步
```

## 创建app
`python manage.py startapp <app_name>`，创建之后需要**注册**`<app_name>`使app生效
在settings.py中的`INSTALLED_APPS`注册该app，实例
为first项目创建一个名为app01的app，
``` 
├─app01
│  │  admin.py          后台管理
│  │  apps.py           app启动类
│  │  models.py         对数据库进行操作
│  │  tests.py          单元测试
│  │  views.py          视图，放置函数
│  │  __init__.py
│  │
│  └─migrations         修改数据库时做记录
│
└─first
    │  asgi.py
    │  settings.py
    │  urls.py
    │  wsgi.py
    └─ __init__.py
``` 

## 启动项目
`python manage.py runserver`

## 静态模板
如果在`settings.py`中配置过`'DIRS': [os.path.join(BASE_DIR,'templates')]`,则根据配置位置寻找templates中的静态网页
缺省情况下在一个app文件夹下添加一个templates目录，放置静态网页（html文件），在app中引入该网页时，根据app的注册顺序进行寻找

### 静态资源
在app目录下创建static目录存放css,js,img资源，以供在静态网页中引入
在网页中引入时，推荐对静态资源路径统一配置(当静态资源位置移动时，只需要**修改settings.py文件中的静态资源路径**即可)，方法如下：
在html网页开头添加`{% load static %}`以供载入
引入时路径可写为(以引入img标签为例)`src="{% static 'img/1.png' %}"`

### 模板语法
从函数中向页面传参，包括变量，列表，字典等数据类型，即为`return render(req,'user_add.html' , {'n1':name,'n2':test_list,'n3':test_dic})`中的render()函数添加第三个参数传参
支持for循环
for循环中有一些内置变量如`{{ forloop.counter }}`整型，计数器,`{{forloop.counter0}}`,`{{forloop.first}}`布尔值
```html
<!-- 输出n3字典中的键和值 -->
{% for k,v in n3.items %}
  <li>{{k}}={{v}}</li>
{% endfor %}
```
if语句，**注意：**`==`两侧需要空格，否则报错
```html
{% if n1 == "ydx" %}
  <h2>n1是ydx</h2>
{% elif n1 == 'ydx2' %}
  <h2>n1是ydx2</h2>
{% else %}
  <h2>n1不是ydx</h2>
{% endif %}
```

## POST请求
django含由csrf_token验证，在提交post请求时需要带上该token否则出错，实例：
```html
<form method='post' action='/login/'>.
  {% csrf_token %}
  <input type="text" name="user" placeholder="用户名">
  <input type="password" value="pwd" placeholder="密码">
  <input type="submit" value="提交">
</form>
```

## 重定向到新页面
使用`return redirect('<新页面网址>')`，将该页面的url返回给浏览器，再由浏览器访问该网站、

## 数据库操作
不需要使用pymysql对mysql进行操作，自带有orm框架
### 创建数据库
在settings.py中修改数据库的配置
```python
DATABASES = {
  'default': {
    'ENGINE': 'django.db.backends.mysql',
    'NAME': BASE_DIR / 'db.sqlite3',
    'USER':'root',
    'PASSWORD':'123456',
    'HOST':'123.',
    'PORT':3306,
  }
}
```

### 创建表
在某个应用(需要该应用提前注册)的models.py中写入一个类，表示一个表（示例中的类表示及其等价的sql语句）
```python
class UserInfo(models.Model):
  name= models.CharField(32)
  password=models.CharField(64)
  age=models.IntegerField()

"""
create table app01_userinfo(
  id bigint auto_increment primary key,
  name varchar(32),
  password varchar(64),
  age int,
)
"""
```
类构建完成后，执行以下语句，以类为基础创建该表，执行一次将重新构建一次
`python manage.py makemigrations`
`python manage.py migrate`
在原表中有数据时增加列，执行第一句代码，可以选择两种方式增加列
可以设置默认值
```python
age=models.IntegerField(default=2)
data=models.IntegerField(null=True,blank=True)
```

### 增删改查
- 添加数据
`UserInfo.objects.create(name='yede',password='124',age=12)`
获取从html传回的数据，进行校验然后在views.py中插入数据库

- 删除数据
`UserInfo.objects.filter(id=3).delete()`删除userinfo表中的id为3的行
`UserInfo.objects.all().delete()`删除userinfo表中所有行

- 查询数据
`UserInfo.objects.all()`获取到一个queryset类型数据，类似于对象组成的列表
`UserInfo.objects.filter(id=1)`筛选符合条件的数据，也是queryset类型
`UserInfo.objects.all(id =1).first()`获取上一条中的第一个数据
一般在views.py中获取到数据，然后传参给html文件，html文件再使用循环输出数据

- 更改数据
`UserInfo.objects.all().update(password=999)`将所有数据的password列改为999

## 模板继承
用于写重复的内容，如导航栏等使用模板较为方便
父，可以有多个代码块供继承
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  {% block css %}{% endblock %}
  <title>Document</title>
</head>
<body>
  <h1>你好，这是导航栏</h1>
  {% block content %}{% endblock %}
</body>
</html>
```
子继承
```html
{% extends 'bar.html' %}
{% block content %}
  <span>这是userbar</span>
{% endblock %}
{% block css %}

{% endblock %}
```

## Form与ModelForm
可以理解为将前端的样式等交给后端解决，form可以为指定字段创建widget(如输入框)，在views.py中实例化对象传递给html；modelform功能与form类似，但它是直接对一个model创建widget
以下为modelform的一个简易实例
在views.py中
```python
class UserModelForm(forms.ModelForm):
  class Meta:
    model=models.UserInfo
    fields=['name','password','age']

def addModelForm(req):
  form =UserModelForm()
  return render(req,'add_use.html',{'form':form})
```
在html中
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <!-- label获取name字段在数据库中的verbose -->
  {{form.name.label}}:{{form.name}}

  {% for field in form %}
    {{field.label}}:{{field}}
  {% endfor %}
</body>
</html>
```
使用modelform后，在html中生成的选择框选项为对象（传递了queryset数据类型），则需要去该对象的定义中重写__str__(self)函数，定义对象的返回字符串

为标签添加样式
```python
class UserModelForm(forms.ModelForm):
  class Meta:
    model=models.UserInfo
    fields=['name','password','age']
  # 为每个标签添加样式
  def __init__(self,*args,**kwargs) -> None:
    super().__init__(*args,**kwargs)
    for name,field in self.fields.items():
      field.widget.attrs={'class':'form-control'}
```