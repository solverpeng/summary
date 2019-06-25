# 使用@WebServlet创建Servlet

`@WebServlet`注解用于在Web应用程序中定义Servlet组件。我们考虑遵循web.xml配置。

```xml
<servlet>
    <servlet-name>viewController</servlet-name>
    <servlet-class>com.logicbig.servlet.ViewController</servlet-class>
    <init-param><param-name>renderer-class-name</param-name>
        <param-value>com.solverpeng.HtmlRenderer</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>

......

<servlet-mapping>
    <servlet-name>viewController</servlet-name>
    <url-pattern>/view/*</url-pattern>
</servlet-mapping>
```

在下面的示例中，我们将使用注解来执行相同的映射。

第一步：准备项目

1. 使用 `maven-archetype-webapp`创建xiang'm。
2. 删除web.xml，我们根本不需要它。
3. 删除`webapp/index.jsp`。
4. 在pom.xml中添加`javax.servlet-api:3.0.1`依赖。
5. 在pom.xml中还添加`tomcat7-maven-plugin`以将其作为嵌入式服务器运行。

第二步：创建一个使用`@WebServlet`注解的servlet类`ViewController`。我们还将指定上面web.xml示例中提到的所有配置。

```java
@WebServlet(name = "viewController", urlPatterns = {"/view/*"},
        initParams = @WebInitParam(name = "renderer-class-name"
                , value = "com.solverpeng.HtmlRenderer"),
        loadOnStartup = 1)
public class ViewController extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req,
                         HttpServletResponse resp) throws ServletException, IOException {

        String renderer = getServletConfig().getInitParameter("renderer-class-name");
        PrintWriter writer = resp.getWriter();
        writer.println("renderer: " + renderer);

        String servletName = getServletConfig().getServletName();
        writer.println("servlet name " + servletName);
    }
}
```

第三步：我们的Web应用程序现在可以使用了。从命令行，转到项目根目录并运行。

```
mvn clean tomcat7:run-war
```

第四步：将以下网址放入浏览器：

```
http://localhost:8080/webservlet-example/view
```

