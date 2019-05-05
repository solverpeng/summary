# DispatcherServlet

`Spring MVC` 和其他 web 框架类似，围绕着一个前控制器设计了一个中央 `Servlet`，即`DispatcherServlet`。



和其他`Servlet`类似，`DispatcherServlet`也需要根据 Servlet 规范声明和映射使用 Java 配置或是配置在 web.xml中。相反的，`DispatcherServlet`使用 Spring 配置发现它需要的各种组件，如：请求映射/视图解析/异常处理等等。

初始化`DispatcherServlet`的两种方式：

通过 `SCI` 进行初始化：javax.servlet.ServletContainerInitializer#onStartup

```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
        ac.register(AppConfig.class);
        ac.refresh();

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(ac);
        ServletRegistration.Dynamic registration = servletContext.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```

通过监听器进行初始化：javax.servlet.ServletContextListener#contextInitialized

```xml
<web-app>
    <display-name>Archetype Created Web Application</display-name>
    
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/app-context.xml</param-value>
    </context-param>
    
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```

这两种方式都依赖于Servlet。

## 层次结构

`DispatcherServlet`需要一个 `WebApplicationContext`进行配置。`WebApplicationContext`用来关联`ServletContext`和`Servlet`。可以通过`RequestContextUtils` 的静态方法来查找`WebApplicationContext`。



对于很多应用来说，有一个单独的`WebApplictionContext`是比较简单和方便的。或者也可以有这样的上下文层次结构，其中一个 `根(root)WebApplicationContext`，在多个 `DispatcherServlet`实例间共享，每个实例都有自己的`子（child）WebApplicationContext`。



`根(root)WebApplicationContext`通常包含基础结构`bean`，如`Services`/`Repositories`等会被多个`Servlet`实例访问。这些基础结构的`bean`可以在特定的`子(clild)WebApplicationContext`进行重写。下图显示了这种关系：

![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/github/java-summary/spring-web-mvc/mvc-context-hierarchy.png)



使用如下两种方式配置父子`WebApplicationContext`上下文层次结构：

通过继承`AbstractAnnotationConfigDispatcherServletInitializer`的方式：

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{RootConfig.class};
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{App1Config.class};
    }

    @Override
    protected String[] getServletMappings() {
        return new String[]{"/app1/*"};
    }
}
```

通过`web.xml`配置：

```xml
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/root-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app1</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/app1-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app1</servlet-name>
        <url-pattern>/app1/*</url-pattern>
    </servlet-mapping>

</web-app>
```

如果一个`子(clild)WebApplicationContext`上下文环境不是必须的，应用只需要配置一个`root`上下文环境，且将 `Servlet`参数`contextConfigLocation`配置为空即可。