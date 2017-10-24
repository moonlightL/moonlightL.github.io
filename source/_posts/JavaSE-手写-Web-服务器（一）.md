---
title: JavaSE 手写 Web 服务器（一）
date: 2017-10-22 00:32:01
tags: Web容器
categories: 后端
---
## 一、背景
某日，在 Java 技术群中看到网友讨论 tomcat 容器相关内容，然后想到自己能不能实现一个简单的 web 容器。于是翻阅资料和思考，最终通过 JavaSE 原生 API 编写出一个简单 web 容器（模拟 tomcat）。在此只想分享编写简单 web 容器时的思路和技巧。

## 二、涉及知识

Socket 编程：服务端通过监听端口，提供客户端连接进行通信。

Http 协议：分析和响应客户端请求。

多线程：处理多个客户端请求。

**用到的都是 JavaSE 的基础知识。**

<!-- more -->
## 三、初步模型

### 3.1 通过 Socket API 编写服务端

服务端的功能：接收客户端发送的的数据和响应数据回客户端。

``` java
package com.light.server;

import java.io.BufferedWriter;
import java.io.IOException;
import java.io.OutputStreamWriter;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.Date;

public class Server {

    private static final String BLANK = " ";
    private static final String RN = "\r\n";
    
    private ServerSocket server;
    
    public static void main(String[] args) {
        Server server = new Server();
        server.start();
    }

    /**
     * 启动服务器
     */
    public void start() {
        try {
            server = new ServerSocket(8080);
            // 接收数据
            this.receiveData();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 接收数据
     */
    private void receiveData() {
        try {
            Socket client = this.server.accept();
            // 读取客户端发送的数据
            byte[] data = new byte[10240];
            int len = client.getInputStream().read(data);
            
            String requestInfo = new String(data,0,len);
            // 打印客户端数据
            System.out.println(requestInfo);
            
            // 响应正文
            String responseContent = "<!DOCTYPE html>" + 
                    "<html lang=\"zh\">" + 
                    "    <head>      " +
                    "        <meta charset=\"UTF-8\">"+
                    "        <title>测试</title>"+
                    "    </head>     "+
                    "    <body>      "+
                    "        <h3>Hello World</h3>"+
                    "    </body>     "+
                    "</html>";
            
            
            StringBuilder response = new StringBuilder();
            // 响应头信息
            response.append("HTTP/1.1").append(BLANK).append("200").append(BLANK).append("OK").append(RN);
            response.append("Content-Length:").append(responseContent.length()).append(RN);
            response.append("Content-Type:text/html").append(RN);
            response.append("Date:").append(new Date()).append(RN);
            response.append("Server:nginx/1.12.1").append(RN);
            response.append(RN);
            // 添加正文
            response.append(responseContent);
            
            // 输出到浏览器
            BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(client.getOutputStream()));
            bw.write(response.toString());
            bw.flush();
            bw.close();
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 关闭服务器
     */
    public void stop() {
        
    }

}

```

启动程序，通过浏览器访问 <http://localhost:8080/login?username=aaa&password=bbb>，结果如下图：

![image](http://ow97db1io.bkt.clouddn.com/webserver-01.jpg)

响应信息与代码中设置的一致。


### 3.2 分析客户端数据

#### 3.2.1 获取 get 方式的请求数据

打开浏览器，通过 get 方式请求 <http://localhost:8080/login?username=aaa&password=bbb> 服务端打印内容如下：

```
GET /login?username=aaa&password=bbb HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.104 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.8
Cookie: SESSION=2b5369d6-9d94-4b54-9ef3-05e47fe63025; JSESSIONID=3B48C7BF26937058A433A29EB2F978BC
```

#### 3.2.2  获取 post 方式的请求数据

编写一个简单的 html 页面，发送 post 请求，

``` html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>测试</title>
</head>
<body>
    <form action="http://localhost:8080/login" method="post">
        <table border="1">
            <tr>
                <td>用户名</td>
                <td><input type="text" name="username"></td>
            </tr>
            <tr>
                <td>密码</td>
                <td><input type="password" name="password"></td>
            </tr>
            <tr>
                <td>爱好</td>
                <td>
                    <input type="checkbox" name="likes" value="1">篮球&nbsp;
                    <input type="checkbox" name="likes" value="2">足球&nbsp;
                    <input type="checkbox" name="likes" value="3">棒球
                </td>
            </tr>
            <tr>
                <td colspan="2" align="center">
                    <input type="submit" value="提交">&nbsp;&nbsp;
                    <input type="reset" value="重置">
                </td>
            </tr>
        </table>
    </form>
</body>
</html>
```

服务端打印内容如下：

```
POST /login HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Content-Length: 41
Cache-Control: max-age=0
Origin: null
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.104 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.8
Cookie: SESSION=2b5369d6-9d94-4b54-9ef3-05e47fe63025; JSESSIONID=3B48C7BF26937058A433A29EB2F978BC

username=aaa&password=bbb&likes=1&likes=2
```

通过分析和对比两种请求方式的数据，我们可以得到以下结论：

共同点：请求方式、请求 URL 和请求协议都是放在第一行。

不同点：get 请求的请求参数与 URL 拼接在一起，而 post 请求参数放在数据的最后一行。

## 四、封装请求和响应

Java 作为面向对象的程序开发语言，封装是其三大特性之一。

通过上文的结论，我们可以将请求数据和响应数据进行封装，让代码更具扩展性和阅读性。

### 4.1 封装请求对象

``` java
public class Request {

    // 常量（回车+换行）
    private static final String RN = "\r\n";
    private static final String GET = "get";
    private static final String POST = "post";
    private static final String CHARSET = "GBK";
    // 请求方式
    private String method = "";
    // 请求 url
    private String url = "";
    // 请求参数
    private Map<String, List<String>> parameterMap;

    private InputStream in;

    private String requestInfo = "";
    
    public Request() {
        parameterMap = new HashMap<>();
    }

    public Request(InputStream in) {
        this();
        this.in = in;
        try {
            byte[] data = new byte[10240];
            int len = in.read(data);
            requestInfo = new String(data, 0, len);
        } catch (IOException e) {
            return;
        }
        // 分析头信息
        this.analyzeHeaderInfo();
    }

    /**
     * 分析头信息
     */
    private void analyzeHeaderInfo() {
        if (this.requestInfo == null || "".equals(this.requestInfo.trim())) {
            return;
        }
        
        // 第一行请求数据： GET /login?username=aaa&password=bbb HTTP/1.1
        
        // 1.获取请求方式
        String firstLine = this.requestInfo.substring(0, this.requestInfo.indexOf(RN));
        int index = firstLine.indexOf("/");
        this.method = firstLine.substring(0,index).trim();
        
        String urlStr = firstLine.substring(index,firstLine.indexOf("HTTP/1.1")).trim();
        String parameters = "";
        if (GET.equalsIgnoreCase(this.method)) {
            if (urlStr.contains("?")) {
                String[] arr = urlStr.split("\\?");
                this.url = arr[0];
                parameters = arr[1];
            } else {
                this.url = urlStr;
            }
            
        } else if (POST.equalsIgnoreCase(this.method)) {
            this.url = urlStr;
            parameters = this.requestInfo.substring(this.requestInfo.lastIndexOf(RN)).trim();
        }
        
        // 2. 将参数封装到 map 中
        if ("".equals(parameters)) {
            return;
        }
        this.parseToMap(parameters);
    }

    /**
     * 封装参数到 Map 中
     * @param parameters
     */
    private void parseToMap(String parameters) {
        // 请求参数格式：username=aaa&password=bbb&likes=1&likes=2
        
        StringTokenizer token = new StringTokenizer(parameters, "&");
        while(token.hasMoreTokens()) {
            // keyValue 格式：username=aaa 或 username=
            String keyValue = token.nextToken();
            String[] kv = keyValue.split("=");
            if (kv.length == 1) {
                kv = Arrays.copyOf(kv, 2);
                kv[1] = null;
            } 
            
            String key = kv[0].trim();
            String value = kv[1] == null ? null : this.decode(kv[1].trim(), CHARSET);
            
            if (!this.parameterMap.containsKey(key)) {
                this.parameterMap.put(key, new ArrayList<>());
            }
            
            this.parameterMap.get(key).add(value);
        }
    }
    
    /**
     * 根据参数名获取多个参数值
     * @param name
     * @return
     */
    public String[] getParameterValues(String name) {
        List<String> values = null;
        if ((values = this.parameterMap.get(name)) == null) {
            return null;
        }
        
        return values.toArray(new String[0]);
    }
    
    /**
     * 根据参数名获取唯一参数值
     * @param name
     * @return
     */
    public String getParameter(String name) {
        String[] values = this.getParameterValues(name);
        if (values == null) {
            return null;
        }
        return values[0];
    }
    
    
    /**
     * 解码中文
     * @param value
     * @param code
     * @return
     */
    private String decode(String value, String charset) {
        try {
            return URLDecoder.decode(value, charset);
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return null;
    }

    public String getUrl() {
        return url;
    }
 }

```


### 4.2 封装响应对象

``` java
public class Response {
    // 常量
    private static final String BLANK = " ";
    private static final String RN = "\r\n";
    // 响应内容长度
    private int len;
    // 存储头信息
    private StringBuilder headerInfo;
    // 存储正文信息
    private StringBuilder contentInfo;
    // 输出流
    private BufferedWriter bw;
    
    public Response() {
        headerInfo = new StringBuilder();
        contentInfo = new StringBuilder();
        len = 0;
    }
    
    public Response(OutputStream os) {
        this();
        bw = new BufferedWriter(new OutputStreamWriter(os));
    }
    
    /**
     * 设置头信息
     * @param code
     */
    private void setHeaderInfo(int code) {
        // 响应头信息
        headerInfo.append("HTTP/1.1").append(BLANK).append(code).append(BLANK);
        
        if ("200".equals(code)) {
            headerInfo.append("OK");
            
        } else if ("404".equals(code)) {
            headerInfo.append("NOT FOUND");
            
        } else if ("500".equals(code)) {
            headerInfo.append("SERVER ERROR");
        }
        
        headerInfo.append(RN);
        headerInfo.append("Content-Length:").append(len).append(RN);
        headerInfo.append("Content-Type:text/html").append(RN);
        headerInfo.append("Date:").append(new Date()).append(RN);
        headerInfo.append("Server:nginx/1.12.1").append(RN);
        headerInfo.append(RN);
    }
    
    /**
     * 设置正文
     * @param content
     * @return
     */
    public Response print(String content) {
        contentInfo.append(content);
        len += content.getBytes().length;
        return this;
    }
    
    /**
     * 设置正文
     * @param content
     * @return
     */
    public Response println(String content) {
        contentInfo.append(content).append(RN);
        len += (content + RN).getBytes().length;
        return this;
    }
    
    /**
     * 返回客户端
     * @param code
     * @throws IOException
     */
    public void pushToClient(int code) throws IOException {
        // 设置头信息
        this.setHeaderInfo(code);
        bw.append(headerInfo.toString());
        // 设置正文
        bw.append(contentInfo.toString());
        
        bw.flush();
    }
    
    
    public void close() {
        try {
            bw.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

改造 Server 类中的 receiveData 方法，将 IO 流操作替换成 Request 和 Response 对象：

``` java
public class Server {

    private ServerSocket server;
    
    public static void main(String[] args) {
        Server server = new Server();
        server.start();
    }

    /**
     * 启动服务器
     */
    public void start() {
        try {
            server = new ServerSocket(8080);
            // 接收数据
            this.receiveData();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 接收数据
     */
    private void receiveData() {
        try {
            Socket client = this.server.accept();
            // 读取客户端发送的数据
            Request request = new Request(client.getInputStream());
            
            // 响应数据
            Response response = new Response(client.getOutputStream());
            response.println("<!DOCTYPE html>")
                    .println("<html lang=\"zh\">")
                    .println("    <head>      ")
                    .println("        <meta charset=\"UTF-8\">")
                    .println("        <title>测试</title>")
                    .println("    </head>     ")
                    .println("    <body>      ")
                    .println("        <h3>Hello " + request.getParameter("username") + "</h3>")// 获取登陆名
                    .println("    </body>     ")
                    .println("</html>");

            response.pushToClient(200);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 关闭服务器
     */
    public void stop() {

    }

}

```

使用 post 请求方式提交表单，返回结果结果如下：

![image](http://ow97db1io.bkt.clouddn.com/webserver-02.jpg)


## 五、多线程
目前，程序启动后每接收一次请求，程序就会运行中断，这样就没法处理下个客户端请求。

因此，我们需要使用多线程处理多个客户端的请求。

创建一个 Runnable 作为请求分发器，处理客户端的请求：

``` java
public class Dispatcher implements Runnable {
    // socket 客户端
    private Socket socket;
    // 请求对象
    private Request request;
    // 响应对象
    private Response response;
    // 响应码
    private int code = 200;
    
    public Dispatcher(Socket socket) {
        this.socket = socket;
        try {
            this.request = new Request(socket.getInputStream());
            this.response = new Response(socket.getOutputStream());
        } catch (IOException e) {
            code = 500;
            return;
        }
        
    }

    @Override
    public void run() {
        this.response.println("<!DOCTYPE html>")
        .println("<html lang=\"zh\">")
        .println("    <head>      ")
        .println("        <meta charset=\"UTF-8\">")
        .println("        <title>测试</title>")
        .println("    </head>     ")
        .println("    <body>      ")
        .println("        <h3>Hello " + request.getParameter("username") + "</h3>")// 获取登陆名
        .println("    </body>     ")
        .println("</html>");

        try {
            this.response.pushToClient(code);
            this.socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        
    }

}

```

改造 Server 类中的 receiveData 方法，将 Request 和 Response 对象传入分发器中处理请求：

``` java
public class Server {

    private ServerSocket server;
    
    private boolean isShutdown = false;
    
    public static void main(String[] args) {
        Server server = new Server();
        server.start();
    }

    /**
     * 启动服务器
     */
    public void start() {
        try {
            server = new ServerSocket(8080);
            // 接收数据
            this.receiveData();
        } catch (IOException e) {
            this.stop();
        }
    }
    
    /**
     * 接收数据
     */
    private void receiveData() {
        try {
            while(!isShutdown) {
                new Thread(new Dispatcher(this.server.accept())).start();
            }
        } catch (IOException e) {
            this.stop();
        }
    }

    /**
     * 关闭服务器
     */
    public void stop() {
        isShutdown = true;
        try {
            this.server.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}

```

现在，不管浏览器发送几次请求，服务端程序都不会中断了。

但是，目前的 web 服务器只能处理一个简单的请求，不管浏览器端发送什么请求，都是只返回 Hello xxx 的页面，无法根据不同请求处理不同业务。

此问题将在下一篇文章中解决。

## 六、参考资料

* <http://www.cnblogs.com/li0803/archive/2008/11/03/1324746.html>

***未完待续。。。。。。***
