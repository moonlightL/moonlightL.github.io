---
title: 使用 Ant 实现远程部署 Tomcat 项目
date: 2017-10-15 10:09:39
tags: [Ant,Tomcat]
categories: 后端
---
## 一、背景
笔者用 Hexo 搭建此博客系统，搭建过程非常方便。但是，有个麻烦的操作就是每次发布文章都需要重新 clean 和 generate。由于笔者使用的是云服务器，使用 Tomcat 运行博客系统，因此每次还得需要挑选新博文相关的文件远程上传到服务器上。为此，笔者通过 ant 实现自动部署项目。

## 二、编码

### 2.1 搭建环境
1) 安装 jdk 和 ant，并配置环境变量。

2) 下载 [catalina-ant.jar](http://www.java2s.com/Code/Jar/c/Downloadcatalinaant6037jar.htm)，并将其添加到 CLASSPATH 中。如下图：

![image](http://ow97db1io.bkt.clouddn.com/ant-1.jpg)

3) 配置远程 Tomcat 连接的账号和密码。读者可以参考 [《Maven 插件实现 Tomcat 热部署》](http://www.extlight.com/2017/10/14/Maven-%E6%8F%92%E4%BB%B6%E5%AE%9E%E7%8E%B0-Tomcat-%E7%83%AD%E9%83%A8%E7%BD%B2/) 文章中的第二节内容设置 Tomcat。

<!-- more -->

### 2.2 目录结构

|--deploy_demo

|----public

|------web.xml

|----build.xml

其中，deploy_demo 为博客相关文件所在的目录，public 为 Hexo 模板生成的的博文源码文件夹， build.xml 为 ant 编译文件，其他文件/文件夹不一一描述。我们需要将 public 文件夹中的所有文件都部署到远程服务器的 Tomcat 中。

**需要注意的是，我们需要手动将 web.xml 文件添加到 public 文件夹中，否则在执行 ant 后，会返回 build failed 的错误。因为 ant 将文件夹打包成 war 时会检测 web.xml 文件**

### 2.3 实现步骤

1) 将远程 tomcat 容器中的项目删除

2) 将 public 文件夹打包成 war 文件

3) 将 war 文件传输到远程 tomcat 容器中

这些步骤都在 build.xml 文件进行配置，内容如下：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project name="deployBlog" default="redeploy" basedir=".">

    <property name="dist.dir" value="${basedir}" />
    <property name="url" value="http://192.168.2.25:8080/manager/text" />
    <property name="username" value="tomcat" />
    <property name="password" value="tomcat" />
    <property name="path" value="/ROOT" />


    <target name="_def_tomcat_tasks">
        <taskdef name="deploy" classname="org.apache.catalina.ant.DeployTask" />
        <taskdef name="list" classname="org.apache.catalina.ant.ListTask" />
        <taskdef name="reload" classname="org.apache.catalina.ant.ReloadTask" />
        <taskdef name="resources" classname="org.apache.catalina.ant.ResourcesTask" />
        <taskdef name="roles" classname="org.apache.catalina.ant.RolesTask" />
        <taskdef name="start" classname="org.apache.catalina.ant.StartTask" />
        <taskdef name="stop" classname="org.apache.catalina.ant.StopTask" />
        <taskdef name="undeploy" classname="org.apache.catalina.ant.UndeployTask" />
    </target>

    <target name="war" description="WebSip manage">
        <war destfile="${dist.dir}/ROOT.war" webxml="${dist.dir}/public/web.xml" >
            <fileset dir="${dist.dir}/public"/>
        </war>
    </target>

    <target name="redeploy" description="Remove and Install web application" depends="_def_tomcat_tasks">
        <antcall target="undeploy" />
        <antcall target="deploy" />
    </target>

    <target name="deploy" description="Install web application" depends="_def_tomcat_tasks,war">
        <deploy url="${url}" username="${username}" password="${password}" path="${path}" war="${dist.dir}/ROOT.war" />
    </target>

    <target name="undeploy" description="Remove web application" depends="_def_tomcat_tasks">
        <undeploy url="${url}" username="${username}" password="${password}" path="${path}" />
    </target>

    <target name="reload" description="Reload web application" depends="_def_tomcat_tasks,war">
        <reload url="${url}" username="${username}" password="${password}" path="${path}" />
    </target>
</project>

```

如果读者想使用上边的配置，只需按照目录结构放置文件和修改远程服务器相关配置即可。


## 三、部署
全部就绪后，只需在当前目录中打开命令行键入: ant 即可。

编译结果如下：

```
E:\demo\ant\deploy_demo>ant
Buildfile: E:\demo\ant\deploy_demo\build.xml

_def_tomcat_tasks:
Trying to override old definition of datatype resources

redeploy:

_def_tomcat_tasks:

undeploy:
 [undeploy] OK - Undeployed application at context path /

_def_tomcat_tasks:

war:
      [war] Building war: E:\demo\ant\deploy_demo\ROOT.war

deploy:
   [deploy] OK - Deployed application at context path /

BUILD SUCCESSFUL
Total time: 2 seconds
```

就这样，项目被打包部署到远程服务器上了。

我们还可以编写脚本将所有步骤一起执行，以下是笔者编写的 bat 脚本：

```
@echo off

echo  =================开始操作=================

E:

cd \demo\ant\deploy_demo

call hexo clean 

call hexo g

call gulp

call ant -f E:\demo\ant\deploy_demo\build.xml

echo =================执行完毕=================

pause
```

脚本中执行的命令根据自己的要求而定。

将这个脚本保存为任意名（如：upload.bat），并在环境变量中设置其所在目录。这样，我们在任意目录打开 CMD 执行 upload.bat 都可以实现项目的自动化部署。


## 四、参考资料
* <http://www.blogjava.net/amigoxie/archive/2007/11/09/159413.html>
* <http://www.cnblogs.com/GloriousOnion/archive/2012/12/18/2822817.html>
