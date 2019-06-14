# DispatcherServlet

`Spring MVC` 和其他 web 框架类似，围绕着一个前控制器设计了一个中央 `Servlet`，即`DispatcherServlet`。

和其他`Servlet`类似，`DispatcherServlet`也需要根据 Servlet 规范声明和映射使用 Java 配置或是配置在 web.xml中。但不同的是，`DispatcherServlet`使用 Spring 配置发现它需要的各种组件，如：请求映射/视图解析/异常处理等等。



## 初始化方式

初始化`DispatcherServlet`的两种方式：

1. 通过 `SCI` 进行初始化：javax.servlet.ServletContainerInitializer#onStartup

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

2. 通过监听器进行初始化：javax.servlet.ServletContextListener#contextInitialized

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

`DispatcherServlet`需要一个 `WebApplicationContext`进行配置。`WebApplicationContext`用来关联`ServletContext`和`Servlet`。可以通过`RequestContextUtils` 的静态方法来获取`WebApplicationContext`。

```java
WebApplicationContext webApplicationContext = RequestContextUtils.findWebApplicationContext(request);   
```



对于很多应用来说，使用一个单独的`WebApplictionContext`是比较简单和方便的。

对于其他一些应用来说，也可以有这样的上下文层次结构，定义为父子容器。其中一个 `根(root)WebApplicationContext`，在多个 `DispatcherServlet`实例间共享，每个实例都有自己的`子（child）WebApplicationContext`。



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



## 特殊的Bean类型

`DispatcherServlet`委托特殊类型的Bean来处理请求和渲染响应。这些特殊类型的 Bean 意味着是被Spring管理的`Object`实例。这些类型通常有内置的实现，但是我们可以自定义其属性并扩展或替换他们。

有如下几种特殊类型的Bean：

| Bean类型                              | 解释                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| HandlerMapping                        | 映射一个请求到处理器，以及一系列拦截器的前置和后置处理。这种映射基于某些标准，细节因`HandlerMapping`实现而异。<br />主要的两个`HandlerMapping`的实现是`RequestMappingHandlerMapping`（支持`@RequestMapping`注解方法）和`SimpleUrlHandlerMapping`（精确的URI路径模式到处理器） |
| HandlerAdapter                        | 无论处理器实际如何调用，帮助`DispatcherServlet`调用一个处理器映射到一个请求。例如，调用带注解的控制器需要解析注解。`HandlerAdaptor`的主要目的是保护`DispatcherServlet`不受此类细节的影响。 |
| HandlerExceptionResolver              | 解决异常的策略，可能将它们映射到处理器，或者是`HTML`错误页面或者其它。 |
| ViewResolver                          | 将处理器返回的逻辑视图名称（基于字符串的视图名称）解析为用于渲染响应的实际视图。 |
| LocaleResolver，LocaleContextResolver | 解析客户端正在使用的区域设置以及可能的时区，以便能够提供国际化视图 |
| ThemeResolver                         | 解决Web应用程序可以使用的主题。                              |
| MultipartResolver                     | 在一些multipart解析库的帮助下，解析`multi-part`请求，如浏览器表单文件上传。 |
| FlashMapManager                       | 存储和检索`input`和`output`的FlashMap，可用于将属性从一个请求传递到另一个请求，通常是通过重定向。 |

## Web MVC 配置

应用程序声明了处理请求所需的特殊Bean列表。`DispatcherServlet`检测每个特殊Bean的`WebApplicationContext`。如果没有匹配的Bean类型，它将使用`DispatcherServlet.properties`中列出的默认类型。



## Servlet 配置

在 Servlet 3.0+ 环境中，可以选择以编程方式配置`Servlet`容器作为替代方法，也可以与`web.xml`文件组合使用。下面的例子展示如何注册一个`DispatcherServlet`：

```java
public class MyWebApplicationInitializer2 implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        XmlWebApplicationContext context = new XmlWebApplicationContext();
        context.setConfigLocation("/WEB-INF/dispatcher-config.xml");

        ServletRegistration.Dynamic registration = servletContext.addServlet("dispatcher", 
                new DispatcherServlet(context));
        registration.setLoadOnStartup(1);
        registration.addMapping("/");
    }
}
```

`WebApplicationInitializer`是SpringMVC提供的接口，确保检测到你的现实，并自动用于初始化任何 Servlet 3容器。`WebApplicationInitializer`的抽象基类`AbstractDispatcherServletInitializer`通过重写方法来指定`servlet`映射和`DispatcherServlet`配置的位置，更容易注册DispatcherServlet`。



使用基于Java的Spring配置应用程序，推荐如下做法：

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

如果使用基于XML的Spring配置，应该继承`AbstractDispatcherServletInitializer`，如下：

```java
public class MyWebAppInitializer2 extends AbstractDispatcherServletInitializer {
    @Override
    protected WebApplicationContext createServletApplicationContext() {
        XmlWebApplicationContext context = new XmlWebApplicationContext();
        context.setConfigLocation("WEB-INF/dispatcher-config.xml");
        return context;
    }

    @Override
    protected String[] getServletMappings() {
        return new String[0];
    }

    @Override
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }
}
```

`AbstractDispatcherServletInitializer`也提供了一种方便的方式添加`Filter`实例，且会自动映射到`DispatcherServlet`，如下：

```java
public class MyWebAppInitializer2 extends AbstractDispatcherServletInitializer {
	// ...

    @Override
    protected Filter[] getServletFilters() {
        return new Filter[] {
                new HiddenHttpMethodFilter(), new CharacterEncodingFilter() };
    }
}
```

每个`Filter`都根据其具体类型添加默认名称，并且自动映射到`DispatcherServlet`。

`AbstractDispatcherServletInitializer` 的 `isAsyncSupported`方法提供了一个单独的设置，在`DispatcherServlet`和映射到它的所有过滤器上启用异步支持。默认情况下，为`true`。



需要需要进一步自定义`DispatcherServlet`本身，则可以覆盖`createDispatcherServlet`方法。



## 请求处理

`DispatcherServlet`按如下步骤处理请求：

1. 在请求中搜索`WebApplicationContext`并将其绑定为控制器和进程中的其他元素可以使用的属性。默认绑定在`DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE`这个KEY下。
2. `locale resolver`绑定到请求，让进程中的元素解析处理请求时要用到的`locale`环境（渲染视图/准备数据等）。如果不需要`locale`解析，则不需要设置`locale resolver`。
3. `theme resolver`绑定到请求，让视图等元素觉得使用哪个`theme`，如果不需要，可以忽略。
4. 如果指定了一个`multipart file resolver`，将检查请求是否是`multiparts`。如果`multiparts`没有找到，请求就会包装成一个`MultipartHttpServletRequest`，以供进程中的其他元素进一步处理。
5. 搜索一个合适的`handler`。如果找到`handler`，与`handler`相关联的一系列执行链路将按顺序执行（前置处理器/后置处理器/控制器等），以准备模型和渲染。另外，对于带有注解的控制器，响应将会被渲染（在`HandlerAdapter`中），替代返回视图。
6. 如果返回模型，则视图被渲染。如果没有模型返回，视图将不会被渲染，因为该请求可能已完成。

`WebApplicationContext`中声明的`HandlerExceptionResolver`用于解决在请求处理期间抛出的异常。这些异常`resolver`允许自定义逻辑以解决异常。



`DispatcherServlet`也支持返回`last-modification-date`，由 Servlet API 指定。



可以自定义`DispatcherServlet`实例添加 Servlet 初始化参数添加到`web.xml`中，支持如下参数：

| 参数                           | 解释                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| contextClass                   | 实现`ConfigurableWebApplicationContext`的类，由此Servlet实例化并本地配置。默认情况下，`XmlWebApplicationContext将被使用`。 |
| contextConfigLocation          | 传递给上下文实例（由contextClass指定）的字符串，用于指示可以在何处可以找到上下文。 |
| namespace                      | `WebApplicationContext`的命名空间。默认是`[servlet-name]-servlet`。 |
| throwExceptionIfNoHandlerFound | 在没有找到一个请求的`handler`的情况下，抛出异常。该异常可以被`HandlerExceptionResolver`捕获，并像处理其他任何方法一样处理异常。<br />默认情况下，设置为`false`，这种情况下，`DispatcherServlet`将响应状态设置为404而不引发异常。 |

## 拦截

所有`HandlerMapping`实现都支持`handler interceptors`，当需要将特定功能应用于某些请求时，这些`handler interceptors`非常有用。拦截器必须实现`org.springframework.web.servlet.HandlerInterceptor`，三个方法提供了足够的灵活性来进行预处理和后处理。

- `preHandle(..)`：在实际处理器被执行之前
- `postHandle(..)`：在实际处理器被调用之后
- `afterCompletion(..)`：当一个请求完成的时候

`preHandle(..)`返回一个布尔值。可以使用这个方法跳出或继续执行一个执行链。当返回`true`，处理器执行链将继续执行。当返回`false`，`DispatcherServlet`会认为拦截器本身已经处理了请求，且不会继续执行其他拦截器，以及在执行链中实际的处理器。

也可以使用各个`HandlerMapping`实现上的setter方法直接注册。

注意，`postHandle(..)`对于`@ResponseBody`和`ResponseEntity`标注的方法不起作用，在`postHandle(..)`方法之前，`HandlerAdapter`写入和提交了响应。这意味着对响应进行任何更改都不会生效，如添加额外的标头。对于这种情况，你可以实现`ResponseBodyAdvice`，并且声明该实现类作为一个`Controller Advice bean`，或者在`RequestMappingHandlerAdapter`上直接配置。



## 异常解析

如果在请求映射或者请求处理的过程中抛出异常，`DispatcherServlet`会委托`HandlerExceptionResolver`链去处理异常，提供解决方案，通常是返回一个错误的响应。

`HandlerExceptionResolver`有如下实现类：

| HandlerExceptionResolver          | 描述                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| SimpleMappingExceptionResolver    | 异常类和错误视图名之间的映射。用于在浏览器应用程序中呈现错误页面。 |
| DefaultHandlerExceptionResolver   | 将Spring MVC引发的异常映射到HTTP状态码。                     |
| ResponseStatusExceptionResolver   | 使用@ResponseStatus注释解析异常，并根据注解中的值将它们映射到HTTP状态码。 |
| ExceptionHandlerExceptionResolver | 在`@Controller`或`@ControllerAdvice`类中调用`@ExceptionHandler`方法来解决异常。 |

可以通过在Spring配置文件中声明多个`HandlerExceptionResolver bean`并根据需要设置其顺序属性来形成异常解析程序链。根据需要设置`order`属性。`order`属性值越高，异常解析器越晚被发现。

`HanlderExceptionResolver`可以返回如下：

- 一个指向错误视图的`ModelAndView`
- 如果在解析程序中处理异常，则返回一个空的`ModelAndView`。
- 如果异常未被解析，则为`null`，以便后续解析器尝试解析，如果异常保留在最后，则允许冒泡到Servlet容器。

MVC Config自动声明内置的解析器，用于默认的Spring MVC异常，包括对`@ResponseStatus`注解的异常，以及对`@ExceptionHandler`方法的支持。可以使用自定义的异常解析器去替代它。



## 错误页面

如果任何`HandlerExceptionResolver`仍然无法解析异常，那么它将会将响应状态设置为错误状态（4xx，5xx），Servlet容器渲染一个默认的错误页面。要自定义容器中默认的错误页面，可以在`web.xml`中声明一个错误页面映射。如下：

```java
<error-page>
	<error-code>404</error-code>
    <location>/error.jsp</location>
</error-page>
```

当异常冒泡到Servlet 容器，有错误状态码时，Servlet容器可以在容器内对配置的 URL 进行 ERROR 调度。如 /error.jsp。然后，`DispatcherServlet`会对其进行处理，可能会将其映射到`@Controller`，该控制器可以实现为带有模型的错误视图名称或者呈现JSON响应。如下例所示：

```java
@RestController
public class ErrorController {

    @RequestMapping(path = "/error")
    public Map<String, Object> handle(HttpServletRequest request) {
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("status", request.getAttribute("javax.servlet.error.status_code"));
        map.put("reason", request.getAttribute("javax.servlet.error.message"));
        return map;
    }

}
```

Servlet API 没有提供一种创建错误页面映射的Java方式。可以同时使用`WebApplicationInitialzer`和最小的`web.xml`配置。



## 视图解析

Spring MVC定义了`ViewResolver`和`View`接口，允许在浏览器中渲染模型，而无需和特定的视图技术联系起来。`ViewResolver`提供了视图名称和实际视图之间的映射关系。`View`解决了在移交给特定视图技术之前准备数据的问题。

`ViewResolver`的层次结构：

| ViewResolver                   | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| AbstractCachingViewResolver    | `AbstractCachingViewResolver`的子类缓存它们解析的视图实例。缓存提高了某些视图技术的性能。通过将cache属性设置为false，可以关闭缓存。此外，如果必须在运行时刷新某个视图（如修改FreeMarker模板时），则可以使用`removeFromCache(string viewname, locale loc)`方法 |
| XmlViewResolver                | `ViewResolver`的实现，它接受用XML编写的配置文件，该配置文件的DTD与Spring的XML bean工厂相同。默认配置文件是`/WEB-INF/views.xml`。 |
| ResourceBundleViewResolver     | `ViewResolver`的实现，它使用由 bundle base name 指定的`ResourceBundle`中的bean定义。对于它支持解析的每个视图，使用属性值为`[viewname].(class)`作为视图类，属性值为`[viewname].url`作为视图URL。 |
| UrlBasedViewResolver           | `ViewResolver`接口的简单实现，它会将逻辑视图名称直接解析为URL而没有显式映射定义。如果逻辑名称以直接的方式与视图资源的名称匹配，则不需要任意映射。 |
| InternalResourceViewResolver   | `UrlBasedViewResolver`的便捷子类，支持`InternalResourceView`和其子类，如`JstlView`和`TilesView`。可以使用`setViewClass(..)` 为该`resolver`生成的所有视图指定视图类。 |
| FreeMarkerViewResolver         | `UrlBasedViewResolver`的子类，支持`FreeMarkerView`及其自定义子类。 |
| ContentNegotiatingViewResolver | 实现`ViewResolver`接口，该接口根据请求文件名或`Accept`标头解析视图。 |

### 视图处理

可以通过声明多个`view resolver`形成`iew resolver`链，如果有必要，可以通过设置`order`属性来制定排序，`order`属性值越高，优先级越低。

`ViewResolver`可以返回空值，表示找不到该视图。但是，对于`JSP`和`InternalResourceViewResolver`，弄清楚JSP释放存在的唯一方法是通过`RequestDispatcher`执行调度。必须始终将`InternalResourceViewResolver`配置为`resolver`链中最后一个。

配置视图解析就像向Spring配置文件中添加`ViewResolver beans`那么简单。MVC 配置文件为`View Resolvers`和无逻辑视图控制器提供了专用的配置API，这些配置API对于不使用控制器逻辑的HTML模板非常有用。

### 重定向

视图名称中的特殊`redirect:`前缀允许执行重定向。`UrlBasedViewResolver`（及其子类）将此识别为需要重定向的指令。视图名称的其余部分是重定向URL。

实际效果等同于控制器返回`RedirectView`，但现在控制器本身可以按照逻辑视图名称进行操作。逻辑视图名（如：`redirect:/myapp/some/resource`）相对于当前servlet上下文重定向，而名称（如：`redirect:https://myhost.com/some/arbitrary/path`）重定向到绝对路径。

需要注意的点是，如果使用`@ResponseStatus`注解标注控制器方法，则注解值优先于`RedirectView`设置的响应状态。

### 转发

对于最终由`UrlBasedViewResolver`及其子类解析的视图名称，视图名称中特殊`forward:`前缀表示转发。

这将创建一个`InternalResoureView`，将会执行`RequestDispatcher#forward`方法。因此此前缀对于`InternalResoureViewResolver`和`InternalResourceView`不太有用，但如果使用其他视图技术，且强制使用Servlet/JSP引擎处理，则此前缀会有帮助。注意，也可以连接多哦视图解析器。



## Multipart 解析器

`org.springframework.web.multipart.MultipartResolver` 用来解决多文件上传的问题。其中一个实现是基于`Apache Commons FileUpload`，另一个是基于`Serlvet 3.0 `多部分请求解析。

启用`multipart handling`，需要在`DispatchaerServlet`所在的Spring配置文件中使用`multipartResolver`的名称声明`MultipartResolver bean`。`DispatcherServlet`检测到它并将其传入请求。当接收到内容类型为`multipart/form-data`且请求方法为`POST`，`resolver`将解析内容并将当前`httpservletrequest`包装为`multipartttpservletrequest`，以提供对已解析部分的访问，同时将它们公开为请求参数。



### Apache Commons FileUpload

要使用`Apache Commons FileUpload`，需要配置一个名为`multipartResolver`，类型为`CommonsMultipartResolver`的Bean。同时需要添加`commons-fileupload`的依赖。

### Servlet 3.0

通过Servlet容器配置启用Servlet 3.0多部分解析。做法如下：

- Java中，在Servlet 注册时设置`MultipartConfigElement `。
- web.xml中，添加一个`<multipart-config>`添加到servlet声明中。

设置`MultipartConfigElement`如下：

```java
public class AppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    // ...

    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {

        // Optionally also set maxFileSize, maxRequestSize, fileSizeThreshold
        registration.setMultipartConfig(new MultipartConfigElement("/tmp"));
    }

}
```

配置完成后，添加名为`MultipartResolver`，类型为`StandardServletMultipartResolver`的Bean。



## 日志

Spring MVC 中DEBUG级别的日志被设计成紧凑的、最小且友好的。它关注的是一次又一次有用的高价值信息，而其他信息仅在调试特定问题时才有用。

TRACE级别的日志遵循和DEBUG相同的原则，但可以用于调试任何问题。此外，一些日志消息可能在TRACE与DEBUG中显示不同的详细程度。

### 日志中敏感数据

DEBUG和TRACE级别的日志可能会记录敏感信息。这就是默认情况下屏蔽请求参数和标头的原因，并且必须通过DispatcherServlet上的enableLoggingRequestDetails属性显式启用它们的完整日志记录。

下面的例子展示在JAVA配置类中如何做：

```java
public class MyInitializer
        extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return ... ;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return ... ;
    }

    @Override
    protected String[] getServletMappings() {
        return ... ;
    }

    @Override
    protected void customizeRegistration(Dynamic registration) {
        registration.setInitParameter("enableLoggingRequestDetails", "true");
    }

}
```














