## JSP页面使用EL表达式取不到请求域中的值

原因：

通过Idea创建的 maven 工程，自动添加的web.xml中的jsp页面头约束版本太低。

解决：

替换 web.xml 中头信息

```xml
<!DOCTYPE web-app PUBLIC
        "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
        "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
</web-app>

替换为

<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://java.sun.com/xml/ns/javaee  
http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
</web-app>
```

## Handler方法添加@ResponseBody后返回值中文乱码

描述：

1. `web.xml` 添加了 `CharacterEncodingFilter` ，且对请求和响应设置了相应的编码字符。
2. 请求头设置：
   - contentType = application/x-www-form-urlencoded
   - characterEncoding = utf-8
   - Accept = text/html,application/xhtml+xml,application/xml
3. 响应头信息：
   - content-type = text/html;charset=ISO-8859-1 
4. 返回的中文内容显示为乱码

原因：

1. 设置了 `CharacterEncodingFilter` ,即设置了 `response.setCharacterEncoding`。对于`response.setCharacterEncoding`有一个前置条件是：必须在`getWriter`执行之前或者`response`被提交之前。使用 @ResponseBody 后，`response.setCharacterEncoding`并没有对这个请求生效。
2. 可以看到的是，响应头中的 content-type 的 MediaType 使用的是请求头中 Accept 第一个Media Type，字符集默认为：ISO-8859-1。

解决：

1. 将请求头 Accept 设置为：application/json;charset=UTF-8
2. 或者在handler方法处@RequestMapping注解添加 produces 属性，值为： application/json;charset=utf-8