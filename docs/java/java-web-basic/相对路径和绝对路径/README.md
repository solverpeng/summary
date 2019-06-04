# 相对路径和绝对路径

##  相对路径不可靠

相对路径都是以当前资源为基准的，但是在转发的时候，这个基准会变，所以说不可靠。

## 绝对路径，以 `/` 开头

### 由浏览器解析

"/"代表当前服务器的主机地址，`http://localhost:8080/`，多用于重定向。

### 由服务器端解析时

"/"代表当前Web应用`http://localhost:8080/webapp/`，相当于 WebContent目录，多用于服务器内部的转发，还有就是web.xml中，url-pattern中的URL地址。

## base标签

作为整个HTML文档中URL地址的相对路径的基准

## 协议

`request.getProtocol()`

## 服务器地址

`request.getServerName()`

## 端口

`request.getServerPort()`

## Web应用的虚拟路径

`request.getContextPath()`

## 动态Base标签

```xml
<base href="http://${pageContext.request.serverName }:${pageContext.request.serverPort}${pageContext.request.contextPath}/" />
```

## 注意事项

1. 格式：```<base href="http://主机地址/Web应用虚拟路径/" />```
2. 写在head标签内——写在所有URL之前
3. base标签指定的基准仅对相对路径有效
4. base标签中的URL地址要以“/”结束

