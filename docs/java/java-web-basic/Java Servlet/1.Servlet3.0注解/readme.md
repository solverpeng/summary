# Servlet 3.0 注解

- 在Servlet 3.0中，javax.servlet.annotation包中引入了几个新的注解。我们以前在web.xml中执行的配置现在可以通过使用注解来完成。

- 现在，我们可以使用没有web.xml的Servlet，Filter和ServletContextListener。

- 这些注解仅用于提供元数据，我们仍然需要扩展相应的类或实现相应的接口。

- 在WEB-INF/classes目录或WEB-INF/lib中的jar中扫描这些注解类。

- 扫描在部署期间完成。

- 如果WEB-INF/lib中有很多jar，那么servlet容器的启动可能会非常慢，因为每个JAR文件中的每个单独的文件都必须被扫描。

- 我们可以通过在web.xml中使用metadata-complete =“true”来关闭在jar包中扫描这些注解类。

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app metadata-complete="true" id="WebApp_ID" version="3.0"...
  ```

  这也将关闭web fragments 的发现。

- web.xml仍可用于有选择地覆盖注解配置。

- 使用注解，我们没有办法配置以下内容：提供欢迎页面列表，定义错误页面以及提供过滤器顺序。我们必须使用web.xml。

- 引入这些注解以提高开发人员的工作效率。
- 具有更好的默认值，并且基于约定优于配置范例。



以下是Servlet 3.0 API中引入的注解

![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/github/java-summary/java-web-basic/javax-servlet-annotation-api.png)

用法如下：

| 注解                                                         | 等同于web.xml           | Component Usage                                              |
| ------------------------------------------------------------ | ----------------------- | ------------------------------------------------------------ |
| `@WebServlet`                                                | `<servlet>`             | 通常扩展`javax.servlet.http.HttpServlet`                     |
| `@WebFilter`                                                 | `<filter>`              | 实现 `javax.servlet.Filter`                                  |
| `@WebInitParam`                                              | `<init-param>`          | Servlet和Filter初始化参数                                    |
| `@WebListener`                                               | `<listener>`            | 实现如下接口: `javax.servlet.ServletContextListener` `javax.servlet.ServletContextAttributeListener` `javax.servlet.ServletRequestListener` `javax.servlet.ServletRequestAttributeListener``javax.servlet.http.HttpSessionListener ``javax.servlet.http.HttpSessionAttributeListener``javax.servlet.http HttpSessionIdListener` `javax.servlet.http HttpSessionActivationListener` |
| `@MultipartConfig`                                           | `<multipart-config>`    | 负责在multipart/form-data请求中上传文件的Servlet类           |
| `@ServletSecurity`<br />`@HttpMethodConstraint`<br />`@HttpConstraint` | `<security-constraint>` | 指定安全性约束的Servlet类                                    |
| `@HandlesTypes`                                              | Not available           | 实现`javax.servlet.ServletContainerInitializer`接口的类。这是Servlet 3.0中引入的一种新的可插拔性机制。 |

