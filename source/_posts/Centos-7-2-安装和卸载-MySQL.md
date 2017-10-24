---
title: Centos 7.2 安装和卸载 MySQL 5.7
date: 2017-10-02 15:14:45
tags: MySQL
categories: 数据库
---
![](http://ow97db1io.bkt.clouddn.com/mysql-1.jpg)


## 一、背景
闲暇之余在虚拟机安装了 Centos 7.2 系统，按照 [《简单安装MySQL（RPM方式）》](http://www.cnblogs.com/moonlightL/p/7265595.html) 这篇文章安装 MySQL ，发现由于包依赖的问题安装失败，于是索性在官网查询相关文档进行 MySQL 的安装。

## 二、安装

### 2.1 下载
本次安装选择 ***[Installing MySQL on Linux Using the MySQL Yum Repository](https://dev.mysql.com/doc/refman/5.7/en/linux-installation-yum-repo.html)*** 方式

在 <https://dev.mysql.com/downloads/repo/yum/> 选择需要安装的文件，笔者选择 MySQL 5.7 版本。

```
wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

<!-- more -->

### 2.2 安装

```
rpm -ivh mysql57-community-release-el7-11.noarch.rpm

yum install mysql-community-server
```

更多安装方式和细节请参照文章末尾的参考资料

### 2.3 登陆

```
# 重启 MySQL 服务
service mysqld restart

# 获取临时的登陆密码
grep 'temporary password' /var/log/mysqld.log

# 根据上一步获取的密码登陆 MySQL 服务端
mysql -uroot -p

```


### 2.4 修改密码

```
ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';

flush privileges;
```

**注意: MySQL 默认安装了 validate_password 插件，它要求设置的密码长度至少为 8 位数，且需要包含至少一个大写字母，一个小写字母，一个数字和一个特殊符号。**

原文如下：

```
MySQL's validate_password plugin is installed by default. 
This will require that passwords contain at least one upper case letter, 
one lower case letter, one digit, and one special character, and that the total password length
is at least 8 characters.
```

## 三、卸载

### 3.1 查看 MySQL 安装的相关信息

```
rpm -qa | grep -i mysql
```

返回结果：

```
[root@localhost ~]# rpm -qa | grep -i mysql
mysql-community-common-5.7.19-1.el7.x86_64
mysql-community-client-5.7.19-1.el7.x86_64
mysql57-community-release-el7-11.noarch
mysql-community-server-5.7.19-1.el7.x86_64
mysql-community-libs-5.7.19-1.el7.x86_64
```

### 3.2 卸载

```
rpm -ev mysql-community-server-5.7.19-1.el7.x86_64

rpm -ev mysql-community-client-5.7.19-1.el7.x86_64

rpm -ev mysql-community-libs-5.7.19-1.el7.x86_64

rpm -ev mysql57-community-release-el7-11.noarch

rpm -ev mysql-community-common-5.7.19-1.el7.x86_64
```

### 3.3 删除残余文件

```
rm -rf /var/lib/mysql

rm -rf /usr/share/mysql

rm -f /var/log/mysqld.log

rm -f /etc/my.cnf
```

## 四、参考资料
* <https://dev.mysql.com/doc/refman/5.7/en/linux-installation.html>
