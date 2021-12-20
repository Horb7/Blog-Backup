---
mathjax: true
title: MySql安装与简单配置
tags:
- MySql
- Linux
categories: 数据库
---

# Mysql 安装和配置

纯小白开始用 mysql ，记录一下安装与配置的环节，便于以后查看。下文的 mysql 版本均为 $8.0$。 Linux版本为Ubuntu 20.04。

## 安装

在Ubuntu 20.04上，默认情况只有最新版本的mysql包含在apt存储库里，所以要先更新服务器的软件包索引：

`sudo apt update`

接下来安装mysql默认包。

`sudo apt install mysql-server`

安装完mysql后，它默认是启动的。



## 配置mysql

### 初始化数据目录

mysql安装完毕后，数据目录必须初始化，使用 `sudo mysql_secure_installation` 来自动初始化数据目录。

之后需要对mysql安装的安全选项做一些修改。

1. 是否安装验证密码插件，用来测试mysql密码的强度。
2. 为mysql root用户设置密码，然后确认。
3. 之后的提示可以按 $enter$ 使用默认值。

### 修改用户身份验证和使用权限

在 mysql8.0 中，mysql root 用户设置为使用默认的 auth_socket 插件进行身份验证，而不是密码。如果要使用外部程序来访问用户，操作会变得繁琐。

我们可以把身份验证修改为使用密码验证。即把 $auth\_socket$ 变成 $mysql\_native\_password$ 模式。

首先进入到mysql里，`sudo mysql`

接下来我们可以查看mysql用户的账号使用的身份验证方式。

`SELECT user, plugin, host FROM mysql.user;`

![image-20211209232523927](/images/db/modify_passwd.png)

由于我这里已经设置了，所以是 $mysql\_native\_password$ 模式。

我们可以使用 $ALTER \ \ USER$ 命令修改root用户。注意要把 $password$ 设置为选择的密码规范（满足一定强度）。

`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password by 'password';` ，这里$password$ 填入你需要更改的密码。

然后需要使用 `FLUSH PRIVILEGES;` 来使服务器重新加载授权表并使新更改生效。

之后我们可以再次使用 `SELECT user, plugin, host FROM mysql.user;` 来验证是否修改了root 用户的登陆模式。

注意，如果使用了密码登录，我们可以使用 `mysql -u root -p` 这样的方式来访问mysql root用户。

#### 如何更改密码规范？

在我们设置密码的时候可能会出现 "mysql: Your password does not satisfy the current policy requirements" 这样的错误，这是因为密码规范比较高，而你设置的密码太简单。

如果要设置简单一点的密码，我们需要更改密码规范：

使用 `show variables like 'validate_password%'` 来查看当前的密码规范。

![](/images/db/modify_passwd2.png)

$policy$ 表示当前密码规范强度（有 $LOW$, $MEDIUM$, $HIGH$ 三种强度）。

$length$ 表示密码至少需要多少长度。

我们可以对这两个属性进行设置：

`set global validate_password.policy=0;`

`set global validate_password.length=1;`

接下来我们再次使用 $ALTER \ \ USER$ 命令即可更改用户密码。

### 修改用户或添加权限

#### 查看用户

如果要查看用户比较少的属性，可以直接使用

`use mysql;`

`SELECT user, host FROM mysql.user`

来查询。

#### 创建新用户

`CREATE USER 'user_name'@'host' IDENTIFIED BY 'password'` 

其中：

user_name 是你创建出来的用户的用户名。

host 表示这个新创建的用户登录模式，$localhost$ 表示只能从本服务器登录，$\%$ 表示可以从远程登录。

password 为新用户的密码，可以不填。

#### 授权用户

`GRANT privileges ON databasename.tablename TO 'username'@'host'`

其中：

privileges 表示赋予的权利，如 select，update，insert，delete等，如果要赋予全部权力，可以使用ALL。

databasename.tablename 表示某个数据库的某个表，如果可以对任意数据库的任意表做操作，可以使用 \*.\* 。

username@host 表示某一个用户。

#### 撤销用户权限

`REVOKE privileges ON dataname.tablename FROM 'username'@'host'`

##### 撤销操作的注意点

**撤销语句必须和之前的授权语句一模一样，否则无法撤销权限。**

#### 删除用户

`DROP USER 'username'@'host'` 

#### 修改用户

`RENAME USER 'name1'@'host1' TO 'name2'@'host2'` 
