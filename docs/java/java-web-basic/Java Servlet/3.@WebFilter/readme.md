# 使用@WebFilter创建Servlet Filter

@WebFilter注解用于在Web应用程序中定义Servlet Filter组件。

Filter动态拦截请求和响应。Filter转换或使用请求或响应中包含的信息。Filter通常不会自己创建响应，而是提供可附加到任何类型的servlet或JSP页面的常规功能。



在这个例子中，我们将使用`@WebFilter`注解创建我们Profiler filter。首先考虑遵循web.xml过滤器配置。

```xml
<filter>
    <filter-name>profiler</filter-name>
    <filter-class>com.logicbig.filter.Profiler</filter-class>
    <init-param>
        <param-name>env</param-name>
        <param-value>dev</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>profiler</filter-name>
    <url-pattern>/search/*</url-pattern>
</filter-mapping>
```

我们将创建基于等效xml配置的注解过滤器。

第一步：准备项目

1. 使用 `maven-archetype-webapp`创建项目。
2. 删除web.xml，我们根本不需要它。
3. 删除`webapp/index.jsp`。
4. 在pom.xml中添加`javax.servlet-api:3.0.1`依赖。
5. 在pom.xml中还添加`tomcat7-maven-plugin`以将其作为嵌入式服务器运行。

第二步：创建一个使用@WebFilter注解的Filter类`Profiler`，还要添加上面web.xml中提到的所有配置。

第三步：使用具有相同url的注解`@WebServlet`创建一个servlet类`SearchController`，以便可以测试我们的Profiler过滤器。

第四步：现在我们将从根文件夹运行我们的Web应用程序：

```
mvn clean tomcat7:run-war
```

第五步：将以下网址放入浏览器：

```
http://localhost:8080/webfilter-example/search
```

