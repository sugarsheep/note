## 说明

## 目录

## mysql5.7解压版安装步骤

### 下载

[mysql5.7压缩版下载链接](https://dev.mysql.com/downloads/mysql/)

1. ![1562491903152](E:\git\note\数据库\images\1562491903152.png)
2. ![1562491938560](E:\git\note\数据库\images\1562491938560.png)
3. ![1562491958965](E:\git\note\数据库\images\1562491958965.png)

### 解压到任意目录

> 我的是：D:\deInstall

### 配置环境变量

#### 添加：MYSQL_HOME

![1562492288428](E:\git\note\数据库\images\1562492288428.png)

#### 编辑path变量

![1562492402208](E:\git\note\数据库\images\1562492402208.png)

### 新建my.ini文件

> 5.7版本没有该文件需要自己新建，内容如下，注意要修改自己的文件地址。
>
> 将my.ini文件放到D:\Program Files\mysql-5.7.23-winx64目录下。

```ini
[mysqld]
port = 3306
basedir=D:/deInstall/mysql-5.7.26-winx64
datadir=D:/deInstall/mysql-5.7.26-winx64/data
max_connections=200
character-set-server=utf8
default-storage-engine=INNODB
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
[mysql]
default-character-set=utf8
```

### 注册mysql服务

> 执行mysqld -install命令进行安装
>
> mysqld --install MySQL --defaults-file="D:\deInstall\mysql-5.7.26-winx64\my.ini"

若执行命令出现如下情况，使用管理员权限执行即可

![1562493207265](E:\git\note\数据库\images\1562493207265.png)

安装成功

![1562493264553](E:\git\note\数据库\images\1562493264553.png)

### 执行mysqld --initialize-insecure --user=mysql命令初始化

> 执行命令是my.ini中的datadir指定的路径不能存在，成功后，会生成data目录并生成root用户

### 启动、停止MySQL。

- 启动mysql服务：net start mysql
- 停止mysql服务：net stop mysql

### 修改默认密码

> 执行"mysqladmin -u root -p password 新密码"命令设置密码，root旧密码为空，直接回车就可以

### 命令行连接mysql

> mysql -u root -p

### 如何移除mysql服务

> mysqld -remove MySQL

### 安装常见问题

#### 无法启动mysql服务

> 原因：服务的可执行文件路径与实际路径不符

![1562495090258](E:\git\note\数据库\images\1562495090258.png)

解决办法：![1562495144937](E:\git\note\数据库\images\1562495144937.png)

![1562495188665](E:\git\note\数据库\images\1562495188665.png)

## java连接mysql连接串

> jdbc:mysql://localhost:3306/mybatis_test?useSSL=false&serverTimezone=UTC
>
> 说明：serverTimezone是mysql8必须设置的

