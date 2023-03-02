---
title: springboot+mysql+vue2尝试记录
date: 2022/9/24 14:14:12
categories: practical-experience
excerpt: 记录作者使用springboot+mysql+vue2完成一次从本地到部署至服务器的完整流程
hide: false
tag: markdown
---
记录一次springboot+mysql+vue2的尝试记录，从前端干到后端，主要使用IDEA,Mysql(后端),VSCode(前端)，先很粗略的走通一下

{% note warning %}
**环境：**windows10，IDEA2022.2.2，MySQL8.0.30
{% endnote %} 

## IDEA使用
在application.properties文件中
```properties
server.compression.enabled=true
spring.devtools.restart.additional-paths=src/main/java

# 静态资源映射
# 表示静态资源的访问路径（缺省为/**）
spring.mvc.static-path-pattern=/images/**

# 文件上传
<!-- 文件的最大大小 -->
logging.logback.rollingpolicy.max-file-size=10MB
spring.data.cassandra.request.throttler.max-requests-size=10MB
```

## 学习笔记
- controller控制器使用
    - @Controller请求页面和数据，返回html页面（前后端不分离）
    - @RestController请求数据
- RequestMapping负责url的路由映射，可添加在controller类上，其参数有
    - value（请求url的路径），
    - method（http的请求方法，如get（获取资源），delete（删除资源），post（新建或更新），put（更新资源）等），method匹配也可用getmapping，putmapping等
    - consumes请求的媒体类型，
    - produces请求的响应类型，
    - params,headers请求的参数和请求头的值
- 参数传递，@RequestParam(value="nickname"，required=fasle)传入nickname参数映射到其后一个参数，required表示是否可选参数，加上注解后表明该参数必须，
- 常见错误，
    - 4开头是浏览器的错误
    - 404路径错误或未编译
    - 405方法（post等）不匹配，参数有误
- 通配符
    - **表示路径后任意层都是该页面
    - *表示路径后一层是该页面
- 拦截器，HandleInterceptor
    - HandleInterceptor接口有三种方法，preHandle,postHandle,afterCompletion
    - 拦截器注册，

Restful API

## Swagger使用
1. 需要在项目中加入依赖
```xml
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>2.9.2</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>2.9.2</version>
    </dependency>
```
2. 在application.properties中添加配置
```properties
spring.mvc.pathmatch.matching-strategy=ant_path_matcher
```

## Mysql使用
1. 在项目中添加依赖
```xml
        <!--        MyBatisPlus依赖-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.2</version>
        </dependency>
        <!--        mysql驱动依赖-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <!--        数据连接池druid-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.20</version>
        </dependency>
```
2. 在application.properties中配置数据库的相关信息
```properties
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=root
mybatis-plus.comfiguration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```
3. 添加@MapperScan注解
```java
@SpringBootAppliation
@MapperScan("com.xx.mapper")
public class MybatisplusDemoApplication {
    public static void main(String[] args) {
        SpringBootAppliation.run(MybatisplusDemoApplication.class,args)
        
    }
    
}
```

## vue使用
v-if是直接不创建对应标签，v-show是创建标签但通过css隐藏（频繁切换使用v-show）
v-for中的key一般取item.id而不能是index
package.json与IDEA中的pom.xml类似
vue3使用create创建组件；vue2使用render
使用了elementui和[第三方图标库](https://fontawesome.dashgame.com/)
使用axios进行网络请求[参考](https://axios-http.com/zh/docs/req_config)，可直接在main.js中导入axios
使用vuerouter`npm vue-router@3`，有子路由，动态路由，路由传参
路由跳转分为声明式和编程式
路由守卫
```vue
router.beforeEach((to,from,next)=>{
    if(to.path==='/main'&&!isAuthenticated){
        next('/login')
    }
    else{
        next()
    }
})
```

## Vuex使用
vuex中的State,Getter,Mutation,Action,Module五个概念
vue2对应安装vuex3`npm install vuex@3`

跨域问题
不同源（同源是指两个页面具有相同的协议、主机和端口号）的客户端脚本在没有明确授权的情况下，不能读写对方资源，可通过CORS解决（后端实现）

## mock使用
用于前端进行接口调用测试，模拟接口
语法规范：
1. 数据模板（Data Template Definition，DTD）
2. 数据占位符（Data Placeholder Definition，DPD）
数据模板的组成：属性名name，生成规则rule，属性值value
`'name|rule': value`
```json
// 七种格式
'name|min-max': value
'name|count': value
'name|min-max.dmin-dmax': value
'name|min-max.dcount': value
'name|count.dmin-dmax': value
'name|count.dcount': value
'name|+step': value
```
1. 属性值为string
```json
//1. 属性值是string

//重复string [min,max]次
'name|min-max':String
//重复string count次
'name|count':String
```

2. 属性值为numebr
```json
//属性值自动加一
'name|+1':number
//生成一个[min,max]之间的数字,与number所填的数字无关
'name|min-max':number
//生成一个整数在[min,max]，小数部分保留[dmin,dmax]位
'name|min-max.dmin-dmax':number
```
3. 属性值为bool
4. 属性值为Object
5. 属性值为array
6. 属性值为functio
7. 属性值为正则表达式regexp

数据占位符
格式：
```
@占位符
@占位符(参数 [， 参数])

例如
@ip,@email,@cname(随机生成中文名)
```

## vue-element-admin
基于vue和element-ui用于后台管理员使用的界面

## 跨域认证JWT
json web token(JWT)是一个token的具体实现方案
分为三部分：Header(json对象，描述jwt的元数据，alg是签名算法，typ是令牌类型),Payload(json对象，存放实际传递的数据),Signatrue(对前两部分的签名，防止数据篡改)

## 云服务器
ECS(Elastic Compute Service)

## 云端环境准备
以CentOS为例
### 1. Xshell远程连接服务器，xftp上传文件
也可以在终端直接使用ssh连接远程服务器
### 2. Mysql安装（用云服务器系统的版本）
卸载CentOS7自带的mariadb
```shell
# 查找
rpm -qa|grep mariadb
# 卸载,卸载版本应对应查找结果
rpm -e mariadb-libs-5.5.52-1.e17.x86_64 --nodeps
```
解压mysql
```shell
# 创建mysql安装包存放点
mkdir /usr/server/mysql
# 解压，-C指定解压的目录
tar xvf mysql-5.7.34-1.el7.x86_64.rpm-bundle.tar -C mysql
```
执行安装
```shell
# 切换到安装目录后安装依赖
yum -y install libaio
yum -y install libncurses*
yum -y install perl perl-devel

# 正式安装
rpm -ivh mysql-community-common-5.7.34-1.e17.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.34-1.e17.x86_64.rpm
rpm -ivh mysql-community-client-5.7.34-1.e17.x86_64.rpm
rpm -ivh mysql-community-server-5.7.34-1.e17.x86_64.rpm
```
启动mysql
```shell
# 启动mysql
systemctl start mysqld.Service
# 查看生成的临时root密码
cat /var/log/mysqld.log | grep password
```
修改初始的随机密码
```shell
# 登录mysql
mysql -u root -p
Enter passsword: # 输入临时密码
# 更新root密码
set global validate_password_policy=0;
set global validate_password_length=1;
set passsword=passsword('root');
```
授予远程连接的权限
```shell
grant all privilage on *.* to 'root' @'%' identified by 'root';
# 刷新
flush privileges;
```
控制命令
```shell
# mysql的启动 状态查看 关闭
systemctl stop mysqld
systemctl status mysqld
systemctl start mysqld

# 设置为开机自启
systemctl enable mysqld
# 查看是否设置成功
systemctl list-unit-files | grep mysqld
```
关闭防火墙(如果是自己的虚拟机)
```shell
# 查看防火墙状态
firewall-cmd --state
# 停止firewall
systemctl stop firewalld.service
# 禁止firewall开机启动
systemctl disable firewalld.service


```

### 3. nginx安装
```shell
yum install epel-release
yum update
yum -y install nginx
```
nginx命令
```shell
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
```
### 4. 配置jdk
下载所需的jdk后解压
```shell
tar -zvxf jdk-8u131-linux-x64.tar.gz
```
添加环境变量
```shell
vim /etc/profile
# 打开文件后在文末添加
export JAVA_HOME=/usr/server/jdk1.8.0_131 # 解压路径（自定义）
export PATH=${JAVA_HOME}/bin:$PATH

# 使修改生效
source /stc/profile
# 测试
java -version
```
## 项目部署
### 前端
```shell
# 构建项目，生成dist文件夹
npm run build
```
将dist文件夹上传云服务器，再配置nginx
修改nginx/conf.d目录下的vue.conf文件（没有则新建）
```shell
server{
    listen 80;
    # 域名
    server_name localhost;

    location / {
        root /usr/app/dist;# 路径自定义
        index index.html;
    }
}
```
### 后端
将java程序打包生成jar包
```shell
# nohup是在后台执行
nohup java -jar xxx.jar > logName.log 2>&1 &
```

## 参考资料
[^1]: [vue文档](https://cn.vuejs.org/guide/introduction.html)
[^2]:[MyBatisPlus文档](https://baomidou.com/pages/24112f/)
[^3]: [elementui文档](https://element.eleme.cn/#/zh-CN/component/table)
[^4]:[favicon图标制作](https://www.logosc.cn/logo/favicon)
[^5]:[小图标库](https://fontawesome.dashgame.com/)
[^6]:[vue-element-admin文档](https://panjiachen.github.io/vue-element-admin-site/zh/guide/)
