---
title: Java 微服务实践 笔记一
date: 2017-05-15 22:22:08
tags: [java,http,微服务]
categories: java
---

## 静态web内容

HTTP请求内容由Web服务器文件系统提供，常见静态Web内容如：HTML、CSS、JS、JPEG、Flash等等。

### 特点

 - 计算类型：I/O类型
 - 交互方式：单一
 - 资源内容：相同（基本）
 - 资源路径：物理路径（文件，目录）
 - 请求方法：GET（主要）

### 标准优化技术

#### 资源变化
- 响应头：Last-Modified（第一次）
- 请求头：If-Modified-Since（第一次）
- Status Code: 403 Not Modified
- 只有响应头，无返回主题，用本地缓存

#### 资源缓存
- 响应头：ETag（相当与Cache Key）
- 请求头：If-None-Match
### 思考
*为什么Java Web Server不是常用Web Server？*

 - 内存占用：类型，分配
 - 垃圾回收：被动回收(调用System.gc()只是建议虚拟去回收)，停顿
 - 并发处理：线程池


## 动态Web内容

与静态Web内容不同，请求内容通过服务器计算而来。

### 特点

 - 计算类型：混合类型（I/O,CPU,内存）
 - 交互方式：丰富（用户输入，客户端特性[user-agent]）
 - 资源内容：多样性
 - 资源路径：逻辑路径（虚拟）
 - 请求方法：GET/HEAD/PUT/DELETE。。

### 常用应用场景
 - 页面渲染
 - 表单交互
 - Ajax(现在通常不处理XML,而是JSON）
 - XML(RSS)
 - JSON/JSONP
 - Web Services(SOAP、WSDL)
 - WebSocket(Web QQ)

### 请求
 - 资源定位（URI）
 - 请求协议（Protocol）
 - 请求方法（Method)
 - 请求参数（Parameter）
 - 请求主体（Body）
 - 请求头（Header)
 - Cookie
除了Body其他都包含在Header里

### 响应
- 响应头
- 响应主体

### 技术/架构演变

 - CGI（Common Gateway Interface)
 - Servlet
 - JSP
 - Model 1 （JSP + Servlet + JavaBeans）
 - Model 2 （MVC）

### Model 2 与 MVC 的细微差异
- Model 2为面向Web服务的架构，MVC则是面向所有应用场景（比如：PC应用，无线应用）
- 相对于MVC，Model 2中Controller细化为Front Controller(FC)和Application Controller(AC)，前者负责路由后者，后者负责跳转视图。
