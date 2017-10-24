---
title: Maven 插件实现 Tomcat 热部署
date: 2017-10-14 11:39:09
tags: [Maven,Tomcat]
categories: 后端
---
## 一、前言
传统的部署项目方式：关闭 web 容器，将项目放入到 web 容器，启动 web 容器这个三个步骤。步骤不多，但是需要手动完成，频繁的操作总会让人心累。为了“解放双手”，实现自动化部署，本篇介绍通过使用 Maven 实现 Tomcat 的热部署。

## 二、准备
> 本次测试使用一台ip为 192.168.2.25 的虚拟机，系统为 centos 7.2，tomcat 使用 8.5 版本。

### 2.1 配置 tomcat 账号密码

为了能安全地连接远程的 tomcat 服务器，我们需要在 tomcat 上配置账号和密码。

启动 tomcat 服务，在浏览器打开 <http://192.168.2.25:8080>，添加右侧的 “Manager App”，如果出现如下情况：

![image](http://ow97db1io.bkt.clouddn.com/maven-1.jpg)

需要修改 conf/tomcat-users.xml 配置文件。

<!-- more -->

```
vim conf/tomcat-users.xml
```

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">
              
  <!-- 添加如下配置 -->
  <role rolename="manger-script"/>
  <role rolename="manager-gui"/>
  <user username="tomcat" password="tomcat" roles="manager-script,manager-gui"/>

</tomcat-users>
```

**注意：添加的是 manager 的权限，而不是 admin**

还需要创建一个配置文件：

在 conf/Catalina/localhost/ 目录下，创建一个名为 manager.xml 的文件，添加如下内容：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<Context privileged="true" antiResourceLocking="false"
         docBase="${catalina.home}/webapps/manager">
             <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />
</Context>
```

重启 tomcat服务，重新进入主页点击  “Manager App”，出现输入账号密码的弹框，则说明配置成功。

## 三、部署项目

### 3.1 添加 maven 插件

在项目的 pom.xml 文件中添加 tomcat 插件

``` xml
<plugins>
    <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.2</version>
        <configuration>
            <url>http://192.168.2.25:8080/manager/text</url>
            <username>tomcat</username>
            <password>tomcat</password>
            <update>true</update>
            <path>/system</path>
            <!-- war文件路径缺省情况下指向target -->
            <!--<warFile>${basedir}/target/${project.build.finalName}.war</warFile>-->
        </configuration>
    </plugin>
</plugins>
```

其中 url 中的 ip 和 port 根据自己的环境而定，其他为固定值。

username 和 password 的值正是 2.1 中，我们为 tomcat 配置的账号和密码。


### 3.2 打包项目
笔者使用 idea 工具。

![image](http://ow97db1io.bkt.clouddn.com/maven-2.jpg)

### 3.3 部署

部署的过程中，请确保 tomcat 服务是开启的。

未部署前，webapps 目录文件情况：

![image](http://ow97db1io.bkt.clouddn.com/maven-3.jpg)

开始部署，双击图中相关菜单项：

![image](http://ow97db1io.bkt.clouddn.com/maven-4.jpg)

如果读者使用的是 Eclipse 工具，可以右键 pom.xml 文件选择 Run As -> Maven build... -> tomcat7:deploy 或 tomcat7:redeploy  　　


部署后：

![image](http://ow97db1io.bkt.clouddn.com/maven-5.jpg)

## 四、参考资料
* <http://www.cnblogs.com/xyb930826/p/5725340.html>
