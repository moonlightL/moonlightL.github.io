---
title: Nginx 快速入门
date: 2017-10-18 21:28:13
tags: [Nginx,中间件]
categories: 后端
---

![image](http://ow97db1io.bkt.clouddn.com/nginx-logo.png)

## 一、前言
Nginx 是一个高性能的 Web 和反向代理服务器, 它具有有很多非常优越的特性。

### 1.1 作为 Web 服务器
相比 Apache，Nginx 使用更少的资源，支持更多的并发连接，体现更高的效率。能够支持高达 50,000 个并发连接数的响应，其使用 epoll and kqueue 作为开发模型。

### 1.2 作为负载均衡服务器
Nginx 既可以在内部直接支持 Rails 和 PHP，也可以支持作为 HTTP代理服务器 对外进行服务。Nginx 用 C 编写, 不论是系统资源开销还是 CPU 使用效率都比 Perlbal 要好的多。

Nginx 安装非常的简单，配置文件 非常简洁（还能够支持perl语法），Bugs 非常少的服务器: Nginx 启动特别容易，并且几乎可以做到 7*24 不间断运行，即使运行数个月也不需要重新启动。

## 二、安装

### 2.1 添加仓库资源

```
vim /etc/yum.repos.d/nginx.repo
```

添加如下内容：

```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```

**注意 baseurl=http://nginx.org/packages/OS/OSRELEASE/$basearch/ 中 OS/OSRELEASE 表示系统和版本号，根据自己情况而定。**

<!-- more -->

### 2.2 查看 yum 资源列表


```
yum list | grep nginx
```

### 2.3 安装

```
yum install nginx
```

### 2.4 查看版本

```
nginx -v # 使用 -V 可以查看 nginx 安装时依赖的模块
```

### 2.5 常用命令

```
# 启动 nginx 服务
nginx

# 重启 nginx 服务
nginx -s reload

# 关闭 nginx 服务
nginx -s quit

# 测试 nginx.conf 配置文件合法性
nginx -t 

```

### 2.6 信号量

```
# 重新加载配置文件
kill HUP `cat /var/run/nginx.pid`

# 重读日志文件
kill USR1 `cat /var/run/nginx.pid`
```

下图为启动 nginx 服务的默认首页：

![image](http://ow97db1io.bkt.clouddn.com/nginx-01.jpg)

## 三、目录/文件介绍

### 3.1 目录介绍

日志相关

* /etc/logrotate.d/nginx : nginx 日志轮转，用于 logrotate 服务的日志切割

* /var/log/nginx：存放请求日志和错误日志

服务配置相关
* /etc/nginx/nginx.conf：主配置文件

* /etc/nginx/conf.d/default.conf：默认加载配置文件

编码相关

* /etc/nginx/koi-utf

* /etc/nginx/koi-win

* /etc/nginx/win-utf

请求类型相关

* /etc/nginx/mime.types：设置 http 协议的 Content-Type 与扩展名对应关系


守护进程相关

* /usr/lib/systemd/system/nginx-debug.service

* /usr/lib/systemd/system/nginx.service

* /etc/sysconfig/nginx

* /etc/sysconfig/nginx-debug


模块相关

* /usr/lib64/nginx/modules

* /etc/nginx/modules

命令相关

* /usr/sbin/nginx

* /usr/sbin/nginx-debug

帮助文档

* /usr/share/doc/nginx-1.12.1

缓存相关

* /var/cache/nginx


### 3.2 配置文件介绍


```
#运行用户
user nginx;

#启动进程,通常设置成和cpu的数量相等
worker_processes  1;

#全局错误日志及PID文件
error_log  /var/log/nginx/error.log warn;

pid        /var/run/nginx.pid;

#工作模式及连接数上限
events {
    #epoll是多路复用IO(I/O Multiplexing)中的一种方式,
    #仅用于linux2.6以上内核,可以大大提高nginx的性能
    use   epoll; 

    #单个后台worker process进程的最大并发链接数    
    worker_connections  1024;

}


http {
    #设定mime类型,类型由mime.type文件定义
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    
    #设定日志格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，
    #对于普通应用，必须设为 on,
    #如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，
    #以平衡磁盘与网络I/O处理速度，降低系统的uptime.
    sendfile     on;
    #tcp_nopush     on;

    #连接超时时间
    keepalive_timeout  65;
    tcp_nodelay     on;

    #开启gzip压缩
    gzip  on;
    gzip_disable "MSIE [1-6].";

    #设定请求缓冲
    client_header_buffer_size    128k;
    large_client_header_buffers  4 128k;


    #设定虚拟主机配置
    server {
        #侦听80端口
        listen    80;
        
        #定义使用 www.extlight.com访问
        server_name  www.extlight.com;

        #设定本虚拟主机的访问日志
        access_log  logs/nginx.access.log  main;

        #默认请求
        location / {
            #定义服务器的默认网站根目录位置
            root   /usr/share/nginx/html;
             
            #定义首页索引文件的名称
            index  index.html index.htm;

        }

        # 定义错误提示页面
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }

    }
}
```

## 四、location 讲解

location 用于匹配客户端发送的 url ，从而定位请求资源。

匹配方式大致分成 3 类：

* location = xxx {}    精准匹配

* location pattern {}  一般匹配

* location ~ pattern {} 正则表达式匹配


### 4.1 案例分析

假设 Nginx 配置文件中的配置如下：

```
location = / {
    [ 配置 A ]
}

location / {
    [ 配置 B ]
}

location /documents/ {
    [ 配置 C ]
}

location ^~ /images/ {
    [ 配置 D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ 配置 E ]
}
```

当请求 <http://192.168.2.25/> 时，会定位到配置 A；

当请求 <http://192.168.2.25/index.html> 时，会定位到配置 B；

当请求 <http://192.168.2.25/documents/document.html> 时，会定位到配置 C；

当请求 <http://192.168.2.25/images/1.gif> 时，会定位到配置 D；

当请求 <http://192.168.2.25/documents/1.jpg> 时，会定位到配置 E。 


## 五、rewrite 重写

使用 Nginx 提供的全局变量或自己设置的变量，结合正则表达式和标志位实现 url 重写以及重定向。

rewrite 使用范围：在 server、location 和 if 中。

rewrite 与 location 的区别：rewrite 是在同一域名内更改获取资源的路径，而 location 是对一类路径做控制访问或反向代理。

### 5.1 rewrite 语法

```
rewrite regex replacement [flag]
```

语法说明：

```
regex：正则表达式
replacement ：重定向位置
flag：标志位
    last : 相当于Apache的[L]标记，表示完成rewrite
    break : 停止执行当前虚拟主机的后续rewrite指令集
    redirect : 返回302临时重定向，地址栏会显示跳转后的地址
    permanent : 返回301永久重定向，地址栏会显示跳转后的地址
```

### 5.2 if 指令语法
如果在 if 中执行 rewrite 操作，需要了解如下判断规则：

```
if(condition){...}
```

判断规则：
```
当表达式只是一个变量时，如果值为空或任何以 0 开头的字符串都会当做 false
直接比较变量和内容时，使用 = 或 !=
~ 正则表达式匹配，~*不区分大小写的匹配，!~ 区分大小写的不匹配
-f 和 !-f 用来判断是否存在文件
-d 和 !-d 用来判断是否存在目录
-e 和 !-e 用来判断是否存在文件或目录
-x 和 !-x 用来判断文件是否可执行
```

### 5.3 案例分析

```
if ($http_user_agent ~ MSIE) {
    rewrite ^(.*)$ /msie/$1 break; # 如果UA包含"MSIE"，rewrite请求到/msie/目录下
} 

if ($http_cookie ~* "id=([^;]+)(?:;|$)") {
    set $id $1; # 如果cookie匹配正则，设置变量$id等于正则引用部分
 } 

if ($request_method = POST) {
    return 405; # 如果提交方法为POST，则返回状态405（Method not allowed）。return不能返回301,302
} 

if ($slow) {
    limit_rate 10k; # 限速，$slow可以通过 set 指令设置
} 

if (!-f $request_filename){
    break;
    proxy_pass  http://127.0.0.1; # 如果请求的文件名不存在，则反向代理到 localhost 。这里的break也是停止rewrite检查
} 

if ($args ~ post=140){
    rewrite ^ http://example.com/ permanent; # 如果query string中包含"post=140"，永久重定向到 example.com
} 

```

常用变量如下：

```
$args             # 这个变量等于请求行中的参数，同 $query_string
$content_length   # 请求头中的 Content-length 字段。
$content_type     # 请求头中的 Content-Type 字段。
$document_root    # 当前请求在 root 指令中指定的值。
$host             # 请求主机头字段，否则为服务器名称。
$http_user_agent  # 客户端 agent 信息
$http_cookie      # 客户端 cookie 信息
$limit_rate       # 这个变量可以限制连接速率。
$request_method   # 客户端请求的动作，通常为 GET 或 POST。
$remote_addr      # 客户端的IP地址。
$remote_port      # 客户端的端口。
$remote_user      # 已经经过 Auth Basic Module 验证的用户名。
$request_filename # 当前请求的文件路径，由 root 或 alias 指令与 URI 请求生成。
$scheme           # HTTP 方法（如http，https）。
$server_protocol  # 请求使用的协议，通常是 HTTP/1.0 或 HTTP/1.1。
$server_addr      # 服务器地址，在完成一次系统调用后可以确定这个值。
$server_name      # 服务器名称。
$server_port      # 请求到达服务器的端口号。
$request_uri      # 包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。
$uri              # 不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。
$document_uri     # 与 $uri 相同。
```

## 六、访问控制

依赖 **[ngx_http_access_module](http://nginx.org/en/docs/http/ngx_http_access_module.html)** 模块。

在 /etc/nginx/nginx.conf 文件末尾中，有一行 include /etc/nginx/conf.d/*.conf; 设置，表示该文件会引入 /etc/nginx/conf.d/ 下的 .conf 结尾的文件。

在 /etc/nginx/conf.d/ 下有一个 default.conf 文件，我们修改其部分内容：

```
server_name  access;

location / {
    root   /usr/share/nginx/html;
    deny   192.168.2.2;
    allow all;
    index  index.html index.htm;
}
```

在 location 中配置了 deny 和 allow，表示除了 192.168.2.2 的 ip，允许其他 ip 访问 nginx。其中，192.168.2.2 为宿主机 ip 地址。


## 七、文件压缩

依赖 **[ngx_http_gzip_module](http://nginx.org/en/docs/http/ngx_http_gzip_module.html)** 模块。

```
location ~* \.(jpg|gif|png|txt|xml)$ {
    gzip on; 
    gzip_http_version 1.1;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript image/jpeg image/gif image/png
    ;
    root /opt/test;
    
}
```
压缩 /opt/test 目录下的资源。可通过浏览器调试工具查看请求返回加载资源文件的大小。


常用配置说明：

```
gzip on|off;  # 是否开启 gzip
gzip_buffers 32 4k|16 8k; # 缓冲，压缩在内存中缓存分成几块，每块大小
gzip_comp_level [1~9]; # 推荐使用 6。压缩级别越高，越浪费 cpu 计算资源
gzip_disable
gzip_min_length 200; # 大于 200 字节就进行文件压缩
gzip_http_version 1.0|1.1; # 使用http/1.1 协议才进行文件压缩
gzip_types text/plain # 设置需要压缩文件的类型
gzip_vary on|off # 是否传输 gzip 压缩标识
```

## 八、静态资源缓存

依赖 [ngx_http_headers_module](http://nginx.org/en/docs/http/ngx_http_headers_module.html) 模块。

```
location ~* \.(jpg|gif|png|txt|xml)$ {
    expires 24h
    root /opt/test;
}
```

向响应头中添加 Cache-Control，设置缓存时间为 24 小时。

语法：

```
expires 60s; # 60 秒

expires 10m; # 10 分钟

expires 2h;  # 2 小时

expires 30d; # 30 天
```


## 九、跨域

依赖 **[ngx_http_headers_module](http://nginx.org/en/docs/http/ngx_http_headers_module.html)** 模块。

```

location ~* \.(jpg|gif|png|txt|xml)$ {
    gzip on;
    gzip_http_version 1.1;
    gzip_comp_level 2;
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript image/jpeg image/gif image/png
    ;
    
    # 设置允许跨域请求
    add_header 'Access-Control-Allow-Origin' '*';
    add_header 'Access-Control-Allow-Credentials' 'true';
    add_header 'Access-Control-Allow-Methods' 'GET, POST,PUT,DELETE, OPTIONS';
    
    root /opt/test;
    
}
```

## 十、防盗链

依赖 **[ngx_http_referer_module](http://nginx.org/en/docs/http/ngx_http_referer_module.html)** 模块。

```
location ~* \.(jpg|gif|png|txt|xml)$ {
     
    valid_referers none blocked 192.168.2.25;

    if ($invalid_referer) {
            return 403;
    }
}
```

只允许 192.168.2.25 的服务器请求资源。

## 十一、反向代理

依赖 **[ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html)** 模块。

```
listen       80;
    
location / {
    proxy_pass http://192.168.2.25:8080;
    proxy_redirect default;
    
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    
    proxy_connect_timeout 30;
    proxy_send_timeout 30;
    proxy_read_timeout 30;
    
    proxy_buffer_size 32k;
    proxy_buffering on;
    proxy_buffers 4 128k;
    proxy_busy_buffers_size 256k;
    proxy_max_temp_file_size 256k;
}
```

8080 端口为 tomcat 启动端口。浏览器访问 <http://192.168.2.25> 后会看到 tomcat 首页。

常用配置说明：

```
proxy_set_header xx xx;         # 设置主机头和客户端真实地址，以便服务器获取客户端真实IP                                
proxy_connect_timeout 30;       # nginx 与后端服务器连接超时时间
proxy_send_timeout 30;          # 后端服务器回传数据时间，时间到中断连接
proxy_read_timeout 30;          # nginx 拂去后端服务器数据时间
proxy_buffering on|off;         # 是否开启缓存
proxy_buffer_size 32k;          # 缓冲区大小
proxy_buffers 4 128k;           # 缓冲区数量和大小
proxy_busy_buffers_size 256k;   # 设置系统很忙时可以使用的 proxy_buffers的大小，官方推荐位proxy_buffers * 2
proxy_temp_file_write_size 32k; # 当后端服务器的响应过大时 Nginx 一次性写入临时文件的数据大小
proxy_max_temp_file_size 256k;  # 每个请求能用磁盘上临时文件最大大小
```



## 十二、负载均衡

依赖 **[ngx_http_upstream_module](http://nginx.org/en/docs/http/ngx_http_upstream_module.html)** 模块。

```
upstream mytomcat {
    server 192.168.2.25:8081;
    server 192.168.2.25:8082;
    server 192.168.2.25:8083;
}

server {
    location / {
        proxy_pass http://mytomcat;
    }
}

```

启动 3 个 tomcat 服务。浏览器访问 <http://192.168.2.25>，nginx 默认通过轮询方式将请求转发给 3 个 tomcat 服务。


## 十三、代理缓存

依赖 **[ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html)** 模块。

```
upstream mytomcat {
    server 192.168.2.25:8081;
    server 192.168.2.25:8082;
    server 192.168.2.25:8083;
}

proxy_cache_path /usr/proxy/cache level=1:2 keys_zone=light_cache:10m max_size=10g inactive=60m use_temp_path=off;

server {
    location / {
        proxy_pass http://mytomcat;
        
        proxy_cache light_cache;
        proxy_cache_valid 200 304 12h;
        proxy_cache_valid any 10m;
        proxy_cache_key $host$uri$is_args$args;
        add_header Nginx-Cache "$upstream_cache_status";
    }
}
```

## 十四、配置 HTTPS

依赖 **[ngx_http_ssl_module](http://nginx.org/en/docs/http/ngx_http_ssl_module.html)** 模块。

需要开发者购买 CA 证书。

```
server {
    listen  443;
    
    ssl on;
    ssl_certificate  /etc/nginx/cert/xxx.pem;
    ssl_certificate_key  /etc/nginx/cert/xxx.key;
    
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
}
```

## 十五、参考资料
* <http://nginx.org/en/linux_packages.html>
* <http://nginx.org/en/docs/>
* <http://www.nginx.cn/doc/index.html>
