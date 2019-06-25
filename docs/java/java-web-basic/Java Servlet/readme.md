# Java Servlet

本教程演示如何使用Servlet 3.x注解来声明Web组件，动态注册Web组件以及Web应用程序的模块化。



## Servlet注解

**Servlet 3.0 注解**

Java Servlet 3.0新的注解在javax.servlet.annotation包中定义。用于描述servlet容器如何扫描这些注解。



## Servlets

**使用@WebServlet创建Servlet**

有关如何使用@WebServlet创建servlet组件的示例。



## Filters

**使用@WebFilter创建Filter**

有关如何使用@WebFilter创建filter组件的示例。



**Servlet - 基于DispatcherType应用filter**

 理解DispatcherType。



## Listeners

**使用@WebListener创建Servlet Listener**

有关如何使用@WebListener创建listener组件的示例。



## File Upload

**使用@MultipartConfig创建文件上传Servlet**

有关如何使用@MultipartConfig创建文件上传servlet组件的示例。



## Servlet Security

**使用@ServletSecurity和HTTP Basic Authentication创建安全的Servlet**

有关如何使用@ServletSecurity创建servlet组件以限制对资源的访问的示例。



**Servlet - 使用HttpServletRequest.authenticate()以编程方式触发身份验证**

使用HttpServletRequest.authenticate()方法触发容器管理的身份验证。



**Servlet - 使用login()和logout()方法实现基于表单的程序安全**

如何在Web应用程序中使用编程安全性。



## Dynamic Registration

**Servlet以及Web组件的程序化注册**

使用Java Servlet 3.0 API对servlet，filter和url pattern进行编程定义。程序化配置的使用案例。



**使用示例项目了解ServletContainerInitializer**

ServletContainerInitializer示例项目。演示如何使用新功能设计基于servlet的框架。



## Web Fragment

**使用示例项目的Servlet Web Fragment**

使用容器管理，基于表单的安全性的示例项目的Servlet Web Fragment简介和概念。



## Asynchronous Support

**Servlet - 异步处理**

示例说明如何在filter和servlet中异步处理请求。



## Cookie

**Servlet - 处理Cookie**

使用javax.servlet.http.Cookie。



## Session

**Servlet - HttpSessionListener用例和示例**

HttpSessionListener示例以及在什么情况下应该使用它们？



**Servlet - HttpSessionBindingListener用例和示例**

通过示例了解HttpSessionBindingListener。



**Servlet - HttpSessionAttributeListener用例和示例**

通过示例了解HttpSessionAttributeListener



**Servlet - 会话跟踪模式**

Servlet 3.0引入了一种配置/更改容器底层会话跟踪机制的标准方法。



**Servlet - 使用changeSessionId()来防止会话固定攻击**

如何保护servlet免受会话固定攻击？



**Servlet - 在会话超时时重定向到登录页面**

如何在会话超时后重定向/转发到登录页面？



**Servlet - 在会话超时时保持活动用户活动和自动重定向到登录页面**

使用JQuery自动重定向。



## Error Handling

**Servlet - 错误处理**

如何在Java Servlets中使用自定义错误页面？



## Working with Standard Headers

**Servlet - 使用“Last-Modified”和“If-Modified-Since”标头**

如何在Java Servlet中使用'Last-Modified'和'If-Modified-Since'来维护浏览器端缓存？



**Servlet - 使用'ETag'和'If-None-Match'标头**

如何使用'ETag'和'If-None-Match'标头来缓存Java Servlet中未更改的资源？



**Java Servlet - URL重定向**

如何从Java Servlets执行永久和临时URL重定向？



**Servlet - 跨源资源共享(CORS)**

如何在Java Servlet中实现CORS？



**Servlet - 带有Tomcat CorsFilter示例的Servlet**

使用tomcat过滤器(CorsFilter)提供即用型CORS功能



## Other HTTP Request Methods

**Servlet - doPut()示例**

如何处理Servlets中的HTTP PUT请求？



**Servlet - doHead()示例**

如何在Servlets中处理HTTP HEAD请求？



## Implementing OAuth

**Servlet - 使用Google OAuth API登录**

使用Google帐户登录用户，在我们基于servlet的应用程序中将用户身份验证委派给Google。