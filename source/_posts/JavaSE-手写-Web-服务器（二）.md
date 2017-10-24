---
title: JavaSE 手写 Web 服务器（二）
date: 2017-10-22 21:55:28
tags: Web 容器
categories: 后端
---
## 一、背景
在上一篇文章 [《JavaSE 手写 Web 服务器（一）》](http://www.extlight.com/2017/10/22/JavaSE-%E6%89%8B%E5%86%99-Web-%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%88%E4%B8%80%EF%BC%89/) 中介绍了编写 web 服务器的初始模型，封装请求与响应和多线程处理的内容。但是，还是遗留一个问题：如何根据不同的请求 url 去触发不同的业务逻辑。

这个问题将在本篇解决。


## 二、涉及知识

XML：将配置信息写到 XML 文件，解决硬编码问题。

反射：读取 XML 文件配置并实例化对象。

<!-- more -->

## 三、封装控制器
目前手写的 web 容器只能处理一种业务请求，无论发送什么 url 的请求都是只返回一个内容相似的页面。

为了能很好的扩展 web 容器处理不同请求的功能，我们需要将 request 和 response 封装到控制器，让各个业务的控制器处理对应请求和响应。

### 3.1 抽离控制器

创建一个父类控制器：

``` java
public class Servlet {
    
    public void service(Request request, Response response) {
        doGet(request,response);
        doPost(request,response);
    }

    protected void doGet(Request request, Response response) {
        
    }
    
    protected void doPost(Request request, Response response) {
        
    }
}
```

父类中使用了模板方法的设计模式，将业务处理的行为交给子类去处理。

当我们需要一个控制器去处理登陆请求时，我们创建一个 LoginServlet 类去继承 Servlet 并重写 doGet 或 doPost 方法：


``` java
public class LoginServlet extends Servlet {

    @Override
    protected void doPost(Request request, Response response) {
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
    }

}
```

如果需要处理注册请求，创建一个 RegisterServlet 类继承 Servlet 并重写 doGet 或 doPost 方法即可。

### 3.2 创建 web 容器上下文

为了能更好的管理多个控制器实例以及请求 url 与控制器的对应关系，我们需要一个类对其封装和管理。

``` java
/**
 *  web 容器上下文
 * @author Light
 *
 */
public class ServletContext {

    // 存储处理不同请求的 servlet
    // servletName -> servlet 子类
    private Map<String,Servlet> servletMap;
    
    // 存储请求 url 与 servlet 的对应关系
    // 请求 url -> servletName
    private Map<String,String> mappingMap;
    
    public ServletContext() {
        this.servletMap = new HashMap<String, Servlet>();
        this.mappingMap = new HashMap<String, String>();
    }

    public Map<String, Servlet> getServletMap() {
        return servletMap;
    }

    public void setServletMap(Map<String, Servlet> servletMap) {
        this.servletMap = servletMap;
    }

    public Map<String, String> getMappingMap() {
        return mappingMap;
    }

    public void setMappingMap(Map<String, String> mappingMap) {
        this.mappingMap = mappingMap;
    }
    
}

```

### 3.3 创建配置类
虽然有了 web 容器上下文，但是 web 容器上下文并不是一开始就存放配置信息的。配置信息在 web 容器启动时被注册或写入到上下文中，因此需要一个管理配置的类负责该操作：

``` java
/**
 * 配置文件类
 * @author Light
 *
 */
public class WebApp {

    private static ServletContext context;
    
    static {
        context = new ServletContext();
        
        Map<String,Servlet> servletMap = context.getServletMap();
        Map<String,String> mappingMap = context.getMappingMap();
        
        // 注册 登陆控制器
        servletMap.put("login", new LoginServlet());
        mappingMap.put("/login", "login");
        
        // 如有更多请求需要处理，在此配置对应的控制器即可
    }
    
    /**
     *  通过请求 url 获取对应的 Servlet 对象
     * @param url
     * @return
     */
    public static Servlet getServlet(String url) {
        if (url == null || "".equals(url.trim())) {
            return null;
        }
        
        String servletName = context.getMappingMap().get(url);
        Servlet servlet = context.getServletMap().get(servletName);
        
        return servlet;
    }
}

```

改造 Dispatcher 的 run 方法，从 WebApp 类中获取控制器实例：

``` java
@Override
public void run() {
    // 获取控制器
    Servlet servlet = WebApp.getServlet(this.request.getUrl());
    // 处理请求
    servlet.service(request, response);
    try {
        this.response.pushToClient(code);
        this.socket.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
    
}
```

## 四、解决硬编码问题

在 WebApp 类中，我们配置了 LoginServlet 类以及相关信息，这种写法属于硬编码，且这个两个类发生了耦合。

为了解决上述问题，我们可以将 LoginServlet 类的配置写到一个 xml 文件中，WebApp 类负责读取和解析 xml 文件中的信息并将信息写入到 web 容器上下文中，在 Dispatcher 类中通过反射实例化控制器对象处理请求。

创建 web.xml 配置文件：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app>
    <servlet>
        <servlet-name>login</servlet-name>
        <servlet-class>com.light.controller.LoginServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>login</servlet-name>
        <url-pattern>/login</url-pattern>
    </servlet-mapping>
</web-app>
```

创建两个类用于封装 web.xml  配置文件中的数据，即 &lt;servlet&gt; 和 &lt;servlet-mapping&gt; 相关标签的内容：

``` java
/**
 * 封装 web.xml 中 <servlet> 配置信息
 * @author Light
 *
 */
public class ServletXml {

    private String servletName;
    
    private String servletClazz;

    public String getServletName() {
        return servletName;
    }

    public void setServletName(String servletName) {
        this.servletName = servletName;
    }

    public String getServletClazz() {
        return servletClazz;
    }

    public void setServletClazz(String servletClazz) {
        this.servletClazz = servletClazz;
    }

}


/**
 * 封装 web.xml 中 <servlet-mapping> 配置信息
 * @author Light
 *
 */
public class ServletMappingXml {

    private String servletName;
    
    private List<String> urlPattern = new ArrayList<>();

    public String getServletName() {
        return servletName;
    }

    public void setServletName(String servletName) {
        this.servletName = servletName;
    }

    public List<String> getUrlPattern() {
        return urlPattern;
    }

    public void setUrlPattern(List<String> urlPattern) {
        this.urlPattern = urlPattern;
    }
    
}

```


创建 xml 文件解析器，用于解析 web.xml  配置文件：

``` java
/**
 * xml文件解析器
 * @author Light
 *
 */
public class WebHandler extends DefaultHandler{
    
    private List<ServletXml> servletXmlList;
    private List<ServletMappingXml> mappingXmlList;
    
    private ServletXml servletXml;
    private ServletMappingXml servletMappingXml;
    
    private String beginTag;
    private boolean isMapping;
    
    

    @Override
    public void startDocument() throws SAXException {
        servletXmlList = new ArrayList<>();
        mappingXmlList = new ArrayList<>();
    }

    @Override
    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
        if (qName != null) {
            beginTag = qName;
            
            if ("servlet".equals(qName)) {
                isMapping = false;
                servletXml = new ServletXml();
            } else if ("servlet-mapping".equals(qName)){
                isMapping = true;
                servletMappingXml = new ServletMappingXml();
            }
        }
    }
    
    @Override
    public void characters(char[] ch, int start, int length) throws SAXException {
        if (this.beginTag != null) {
            String str = new String(ch,start,length);
            
            if(isMapping) {
                if("servlet-name".equals(beginTag)) {
                    servletMappingXml.setServletName(str);
                } else if ("url-pattern".equals(beginTag)) {
                    servletMappingXml.getUrlPattern().add(str);
                }
                
            } else {
                if("servlet-name".equals(beginTag)) {
                    servletXml.setServletName(str);
                } else if ("servlet-class".equals(beginTag)) {
                    servletXml.setServletClazz(str);
                }
                
            }
        }
    }

    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {
        if (qName != null) {
            
            if ("servlet".equals(qName)) {
                this.servletXmlList.add(this.servletXml);
            } else if ("servlet-mapping".equals(qName)){
                this.mappingXmlList.add(this.servletMappingXml);
            }
        }
        this.beginTag = null;
    }
    
    public List<ServletXml> getServletXmlList() {
        return servletXmlList;
    }

    public List<ServletMappingXml> getMappingXmlList() {
        return mappingXmlList;
    }

}

```

改造 ServletContext 类：

``` java
// 存储处理不同请求的 servlet
// servletName -> servlet 子类的全名
private Map<String,String> servletMap;
```

即 将 private Map&lt;String,Servlet&gt; servletMap 改成 private Map&lt;String,String&gt; servletMap 。

注意，get 和 set 方法都需要修改。

改造 WebApp 类中注册控制器相关代码：

``` java
/**
 * 配置文件类
 * @author Light
 *
 */
public class WebApp {

    private static ServletContext context;
    
    static {
        context = new ServletContext();
        Map<String,String> servletMap = context.getServletMap();
        Map<String,String> mappingMap = context.getMappingMap();
        
        try {
            SAXParserFactory factory = SAXParserFactory.newInstance();
            SAXParser sax = factory.newSAXParser();
            WebHandler handler = new WebHandler();
            
            // 读取和解析 xml 文件
            sax.parse(Thread
                    .currentThread()
                    .getContextClassLoader()
                    .getResourceAsStream("com/light/server/web.xml"), 
                    handler);
            
            // 注册 servlet
            List<ServletXml> servletXmlList = handler.getServletXmlList();
            for (ServletXml servletXml : servletXmlList) {
                servletMap.put(servletXml.getServletName(), servletXml.getServletClazz());
            }
            
            // 注册 mapping
            List<ServletMappingXml> mappingXmlList = handler.getMappingXmlList();
            for (ServletMappingXml mapping : mappingXmlList) {
                List<String> urls = mapping.getUrlPattern();
                for (String url : urls) {
                    mappingMap.put(url, mapping.getServletName());
                }
            }
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    /**
     *  通过请求 url 获取对应的 Servlet 对象
     * @param url
     * @return
     */
    public static String getServletClazz(String url) {
        if (url == null || "".equals(url.trim())) {
            return null;
        }
        
        String servletName = context.getMappingMap().get(url);
        String servletClazz = context.getServletMap().get(servletName);
        
        return servletClazz;
    }
}

```

改造 Dispatcher 类的 run 方法，通过反射实例化对象：

``` java
@Override
public void run() {
    try {
        // 获取控制器包名
        String servletClazz = WebApp.getServletClazz(this.request.getUrl());
        // 通过反射实例化控制器对象
        Servlet servlet = (Servlet) Class.forName(servletClazz).newInstance();
        // 处理请求
        servlet.service(request, response);
        this.response.pushToClient(code);
        this.socket.close();
    } catch (Exception e) {
        e.printStackTrace();
    }
    
}
```


## 五、总结

手写 web 容器的内容到此结束。

文章中主要介绍编写 web 容器的大致流程，代码中还有许多地方需要考虑（过滤器、监听器、日志打印等）和优化，仅作为笔者对 web 容器的理解与分享，并不是为了教读者重复造轮子。

**学习东西要知其然，更要知其所以然。**

## 六、源码

* [源码下载](https://github.com/moonlightL/httpServer)
