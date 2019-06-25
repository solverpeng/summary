# MVC配置

MVC JAVA配置和MVC XML命名空间配置提供适用于大多数应用程序的默认配置，以及用于自定义它的配置API。

## 启用MVC配置

在JAVA配置中，可以使用@EnableWebMvc注解启用MVC配置，如以下示例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfig {
}
```

在XML配置中，可以使用`<mvc:annotation-driven>`元素来启用MVC配置，如以下示例所示：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven/>

</beans>
```

前面的示例注册了许多Spring MVC基础结构BEAN，并适应类路径上可用的依赖项（例如，JSON，XML等的有效负载转换器）。



## MVC配置API

在JAVA配置中，可以实现WebMvcConfigurer接口，如以下示例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    // Implement configuration methods...
}
```

在XML中，可以检查`<mvc:annotation-driven/>`的属性和子元素。可以查看Spring MVC XML模式或使用IDE的代码完成功能来发现可用的属性和子元素。



## 类型转换

默认格式化，包含了Number和Date类型，包括对@NumberFormat和@DateTimeFormat注解的支持。如果类路径中存在Joda-Time，则还会包含对Joda-Time格式库的完全支持。



在JAVA配置中，可以注册自定义的 Formatter 和 Converter，如以下示例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        // ...
    }
}
```

以下示例显示如何在XML中实现相同的配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven conversion-service="conversionService"/>

    <bean id="conversionService"
            class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="org.example.MyConverter"/>
            </set>
        </property>
        <property name="formatters">
            <set>
                <bean class="org.example.MyFormatter"/>
                <bean class="org.example.MyAnnotationFormatterFactory"/>
            </set>
        </property>
        <property name="formatterRegistrars">
            <set>
                <bean class="org.example.MyFormatterRegistrar"/>
            </set>
        </property>
    </bean>

</beans>
```

> 关于何时使用FormatterRegistrar实现的更多信息，请参阅FormatterRegistrar SPI和FormattingConversionServiceFactoryBean。



## 验证

默认情况下，如果类路径存在Bean Validation（如Hibernate Validator），则LocalValidatorFactoryBean将注册为全局Validator，以便与控制器方法参数一起使用@Valid和Validated。



在JAVA配置中，可以自定义全局Validator实例，如以下示例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public Validator getValidator(); {
        // ...
    }
}
```

以下示例显示如何在XML中实现相同的配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven validator="globalValidator"/>

</beans>
```

请注意，还可以在本地注册Validator实现，如以下示例所示：

```java
@Controller
public class MyController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addValidators(new FooValidator());
    }

}
```

> 如果需要在某处注入LocalValidatorFactoryBean，请创建一个bean并使用@Primary标记它，以避免与MVC配置中声明的那个冲突。



## 拦截器

在JAVA配置中，可以注册拦截器以拦截传入的请求，如以下示例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LocaleChangeInterceptor());
        registry.addInterceptor(new ThemeChangeInterceptor()).addPathPatterns("/**").excludePathPatterns("/admin/**");
        registry.addInterceptor(new SecurityInterceptor()).addPathPatterns("/secure/*");
    }
}
```

以下示例显示如何在XML中实现相同的配置：

```xml
<mvc:interceptors>
    <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"/>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <mvc:exclude-mapping path="/admin/**"/>
        <bean class="org.springframework.web.servlet.theme.ThemeChangeInterceptor"/>
    </mvc:interceptor>
    <mvc:interceptor>
        <mvc:mapping path="/secure/*"/>
        <bean class="org.example.SecurityInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```



## Content Types

可以配置Spring MVC如何根据请求确定所请求的媒体类型（例如，Accept标头，URL路径扩展，查询参数等）。



默认情况下，首先检查URL路径扩展 - 将json，xml，rss和atom注册为已知扩展（取决于类路径依赖性）。第二次检查Accept标头。



请考虑将这些默认值更改为仅接受标头，如果必须使用基于URL的内容类型解析，请考虑使用查询参数策略而不是路径扩展。有关详细信息，请参阅后缀匹配和后缀匹配以及RFD。



在JAVA配置中，可以自定义请求的内容类型解析，如以下示例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer.mediaType("json", MediaType.APPLICATION_JSON);
        configurer.mediaType("xml", MediaType.APPLICATION_XML);
    }
}
```

以下示例显示如何在XML中实现相同的配置：

```xml
<mvc:annotation-driven content-negotiation-manager="contentNegotiationManager"/>

<bean id="contentNegotiationManager" class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
    <property name="mediaTypes">
        <value>
            json=application/json
            xml=application/xml
        </value>
    </property>
</bean>
```



## Message Converters

可以通过重写configureMessageConverters()（替换Spring MVC创建的默认转换器）或覆盖extendMessageConverters()来自定义Java配置中的HttpMessageConverter（自定义默认转换器或将其他转换器添加到默认转换器）。



以下示例使用自定义的ObjectMapper而不是默认的ObjectMapper添加XML和Jackson JSON转换器：

```java
@Configuration
@EnableWebMvc
public class WebConfiguration implements WebMvcConfigurer {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder()
                .indentOutput(true)
                .dateFormat(new SimpleDateFormat("yyyy-MM-dd"))
                .modulesToInstall(new ParameterNamesModule());
        converters.add(new MappingJackson2HttpMessageConverter(builder.build()));
        converters.add(new MappingJackson2XmlHttpMessageConverter(builder.createXmlMapper(true).build()));
    }
}
```

在前面的例子中，Jackson2ObjectMapperBuilder用于为MappingJackson2HttpMessageConverter和MappingJackson2XmlHttpMessageConverter创建一个通用配置，并启用缩进，自定义日期格式，以及jackson-module-parameter-names的注册，这增加了对访问参数名称的支持（Java 8中添加的功能）。



此构建器自定义Jackson的默认属性，如下所示：

- DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES已禁用。
- MapperFeature.DEFAULT_VIEW_INCLUSION已禁用。

如果在类路径中检测到它们，它还会自动注册以下众所周知的模块：

- jackson-datatype-jdk7：支持Java 7类型，例如java.nio.file.Path。
- jackson-datatype-joda：支持Joda-Time类型
- jackson-datatype-jsr310：支持Java 8 Date和Time API类型。
- jackson-datatype-jdk8：支持其他Java 8类型，例如Optional。



> 使用Jackson XML支持启用缩进除了jackson-dataformat-xml之外还需要woodstox-core-asl依赖。

其他可用的Jackson模块：

- jackson-datatype-money：支持javax.money类型（非官方模块）。
- jackson-datatype-hibernate：支持特定于Hibernate的类型和属性（包括延迟加载方面）。

以下示例显示如何在XML中实现相同的配置：

```xml
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper" ref="objectMapper"/>
        </bean>
        <bean class="org.springframework.http.converter.xml.MappingJackson2XmlHttpMessageConverter">
            <property name="objectMapper" ref="xmlMapper"/>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>

<bean id="objectMapper" class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean"
      p:indentOutput="true"
      p:simpleDateFormat="yyyy-MM-dd"
      p:modulesToInstall="com.fasterxml.jackson.module.paramnames.ParameterNamesModule"/>

<bean id="xmlMapper" parent="objectMapper" p:createXmlMapper="true"/>
```



## 视图控制器

这是定义ParameterizableViewController的快捷方式，该方法在调用时立即转发到视图。如果在视图生成响应之前没有要执行的Java控制器逻辑，则可以在静态情况下使用它。



以下JAVA配置示例将请求转发给名为home的视图：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("home");
    }
}
```

以下示例与前面的示例实现相同的功能，但使用XML，使用`<mvc:view-controller>`元素：

```xml
<mvc:view-controller path="/" view-name="home"/>
```



## 视图解析器

MVC配置简化了视图解析器的注册。

以下Java配置示例使用JSP和Jackson作为JSON呈现的默认视图来配置内容协商视图解析：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.enableContentNegotiation(new MappingJackson2JsonView());
        registry.jsp();
    }
}
```

以下示例显示如何在XML中实现相同的配置：

```xml
<mvc:view-resolvers>
    <mvc:content-negotiation>
        <mvc:default-views>
            <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
        </mvc:default-views>
    </mvc:content-negotiation>
    <mvc:jsp/>
</mvc:view-resolvers>
```

但请注意，FreeMarker，Tiles，Groovy Markup和脚本模板也需要配置底层视图技术。

MVC名称空间提供专用元素。以下示例适用于FreeMarker：

```xml
<mvc:view-resolvers>
    <mvc:content-negotiation>
        <mvc:default-views>
            <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
        </mvc:default-views>
    </mvc:content-negotiation>
    <mvc:freemarker cache="false"/>
</mvc:view-resolvers>

<mvc:freemarker-configurer>
    <mvc:template-loader-path location="/freemarker"/>
</mvc:freemarker-configurer>
```

在Java配置中，您可以添加相应的Configurer Bean，如以下示例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.enableContentNegotiation(new MappingJackson2JsonView());
        registry.freeMarker().cache(false);
    }

    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer() {
        FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        configurer.setTemplateLoaderPath("/freemarker");
        return configurer;
    }
}
```



## 静态资源

此选项提供了一种从基于资源的位置列表中提供静态资源的便捷方法。

在下一个示例中，给定以/resources开头的请求，相对路径用于在Web应用程序根目录下或/static下的类路径上查找和提供相对于/public的静态资源。资源将在未来一年内到期，以确保最大限度地使用浏览器缓存并减少浏览器发出的HTTP请求。还会评估Last-Modified标头，如果存在，则返回304状态代码。



以下清单显示了如何使用Java配置执行此操作：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
            .addResourceLocations("/public", "classpath:/static/")
            .setCachePeriod(31556926);
    }
}
```

以下示例显示如何在XML中实现相同的配置：

```xml
<mvc:resources mapping="/resources/**"
    location="/public, classpath:/static/"
    cache-period="31556926" />
```

资源处理程序还支持一系列ResourceResolver实现和ResourceTransformer实现，可以使用它们创建工具链以使用优化的资源。



可以将VersionResourceResolver用于基于从内容，固定应用程序版本或其他计算的MD5哈希的版本化资源URL。ContentVersionStrategy（MD5哈希）是一个不错的选择 - 有一些值得注意的例外，例如与模块加载器一起使用的JavaScript资源。



以下示例显示如何在Java配置中使用VersionResourceResolver：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/public/")
                .resourceChain(true)
                .addResolver(new VersionResourceResolver().addContentVersionStrategy("/**"));
    }
}
```

以下示例显示如何在XML中实现相同的配置：

```xml
<mvc:resources mapping="/resources/**" location="/public/">
    <mvc:resource-chain resource-cache="true">
        <mvc:resolvers>
            <mvc:version-resolver>
                <mvc:content-version-strategy patterns="/**"/>
            </mvc:version-resolver>
        </mvc:resolvers>
    </mvc:resource-chain>
</mvc:resources>
```

然后，可以使用ResourceUrlProvider重写URL并应用完整的解析器和转换器链 - 例如，插入版本。MVC配置提供了ResourceUrlProvider Bean，以便可以将其注入其他bean。还可以使用针对Thymeleaf，JSP，FreeMarker和其他具有依赖于HttpServletResponse #creditURL的URL标记的ResourceUrlEncodingFilter使重写透明。



请注意，在使用EncodedResourceResolver（例如，用于提供gzip或brotli编码的资源）和VersionResourceResolver时，必须按此顺序注册它们。这可确保始终基于未编码的文件可靠地计算基于内容的版本。



WebJars也通过WebJarsResourceResolver支持，当类路径中存在org.webjars：webjars-locator-core库时，WebJarsResourceResolver会自动注册。解析器可以重写URL以包含jar的版本，也可以匹配没有版本的传入URL - 例如，从/jquery/jquery.min.js到/jquery/1.2.0/jquery.min.js。



## 默认Servlet

Spring MVC允许将DispatcherServlet映射到`/`（从而覆盖容器的默认Servlet的映射），同时仍然允许容器的默认Servlet处理静态资源请求。它配置DefaultServletHttpRequestHandler，其URL映射为 `/**`，并且相对于其他URL映射具有最低优先级。



此处理程序将所有请求转发到默认Servlet。因此，它必须按所有其他URL HandlerMappings的顺序保持最后。如果您使用`<mvc:annotation-driven>`就是这种情况。或者，如果设置自己的自定义HandlerMapping实例，请确保将其order属性设置为低于DefaultServletHttpRequestHandler的值，即Integer.MAX_VALUE。



以下示例显示如何使用默认设置启用该功能：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```

以下示例显示如何在XML中实现相同的配置：

```xml
<mvc:default-servlet-handler/>
```

覆盖`/`Servlet映射的警告是必须通过名称而不是路径来检索默认Servlet的RequestDispatcher。DefaultServletHttpRequestHandler尝试使用大多数主要Servlet容器的已知名称列表在启动时自动检测容器的默认Servlet（包括Tomcat，Jetty，GlassFish，JBoss，Resin，WebLogic和WebSphere）。如果使用不同的名称自定义配置默认Servlet，或者在默认Servlet名称未知的情况下使用其他Servlet容器，那么你必须显式提供默认的Servlet名称，如下例所示：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable("myCustomDefaultServlet");
    }

}
```

以下示例显示如何在XML中实现相同的配置：

```xml
<mvc:default-servlet-handler default-servlet-name="myCustomDefaultServlet"/>
```



## 路径匹配

可以自定义与路径匹配和URL处理相关的选项。有关各个选项的详细信息，请参阅PathMatchConfigurer javadoc。



下面的示例演示了如何在Java配置中自定义路径匹配：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        configurer
            .setUseSuffixPatternMatch(true)
            .setUseTrailingSlashMatch(false)
            .setUseRegisteredSuffixPatternMatch(true)
            .setPathMatcher(antPathMatcher())
            .setUrlPathHelper(urlPathHelper())
            .addPathPrefix("/api",
                    HandlerTypePredicate.forAnnotation(RestController.class));
    }

    @Bean
    public UrlPathHelper urlPathHelper() {
        //...
    }

    @Bean
    public PathMatcher antPathMatcher() {
        //...
    }

}
```

以下示例显示如何在XML中实现相同的配置：

```xml
<mvc:annotation-driven>
    <mvc:path-matching
        suffix-pattern="true"
        trailing-slash="false"
        registered-suffixes-only="true"
        path-helper="pathHelper"
        path-matcher="pathMatcher"/>
</mvc:annotation-driven>

<bean id="pathHelper" class="org.example.app.MyPathHelper"/>
<bean id="pathMatcher" class="org.example.app.MyPathMatcher"/>
```



## 高级Java配置

@EnableWebMvc导入DelegatingWebMvcConfiguration，其中：

- 为Spring MVC应用程序提供默认的Spring配置
- 检测并委派给WebMvcConfigurer实现以自定义该配置。

对于高级模式，可以删除@EnableWebMvc并直接从DelegatingWebMvcConfiguration扩展而不是实现WebMvcConfigurer，如以下示例所示：

```java
@Configuration
public class WebConfig extends DelegatingWebMvcConfiguration {

    // ...

}
```

可以在WebConfig中保留现有方法，但现在也可以从基类覆盖bean声明，并且仍然可以在类路径上拥有任意数量的其他WebMvcConfigurer实现。



## 高级的XML配置

MVC命名空间没有高级模式。如果需要在bean上自定义一个无法更改的属性，则可以使用Spring ApplicationContext的BeanPostProcessor生命周期钩子，如以下示例所示：

```java
@Component
public class MyPostProcessor implements BeanPostProcessor {

    public Object postProcessBeforeInitialization(Object bean, String name) throws BeansException {
        // ...
    }
}
```

请注意，需要将MyPostProcessor声明为BEAN，可以在XML中显式声明，也可以通过`<component-scan />`声明来检测它。



----------

## @EnableWebMvc

将此注解添加到@Configuration类可从WebMvcConfigurationSupport导入Spring MVC配置，WebMvcConfigurationSupport是提供MVC Java配置背后的配置主类。

```java
@EnableWebMvc
@Configuration
public class MyWebConfig {
 .....
}
```

如果不使用这个注解，可能最初不会注意到任何差异，但是 content-type和accept header，内容协议将不会生效。

> 这看起来像我们旧的AnnotationMethodHandlerAdapter实现和3.1+ RequestMethodHandlerAdapter之间的差异之一。实际上，出于所有实际目的，您应该始终使用@EnableWebMvc，这将导致我们使用现代基础设施。为了向后兼容基于2.5和3.0的应用程序，如果没有给出明确的配置，DispatcherServlet仍然使用旧的变体。请注意，这将从5.0开始简化，然后删除旧的基础架构。



如果您使用的是基于XML的配置，那么使用`<mvc:annotation-driven/>`作为@EnableWebMvc的替代方案。@EnableWebMvc和`<mvc:annotation-driven/>`具有相同的目的，混合使用在某些情况下不起作用。

以下是@EnableWebMvc片段：

```java
package org.springframework.web.servlet.config.annotation;
 ...
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```

如上所示，EnableWebMvc导入DelegatingWebMvcConfiguration，它是WebMvcConfigurationSupport的子类。

### 如何自定义配置

要自定义@EnableWebMvc导入的配置，我们应该扩展类WebMvcConfigurerAdapter并覆盖我们想要进行相关定制的方法。我们的扩展WebMvcConfigurerAdapter方法在配置阶段从WebMvcConfigurationSupport回调。请注意，WebMvcConfigurerAdapter自5.0以来已被弃用，因此我们应该另外实现WebMvcConfigurer。



如果重写WebMvcConfigurer对我们不起作用并且我们想要进行一些高级配置，那么我们不应该使用@EnableWebMvc注解。在这种情况下，我们应该直接从WebMvcConfigurationSupport或DelegatingWebMvcConfiguration扩展我们的@Configuration类，并有选择地覆盖包括用@Bean注释的工厂方法的方法。

See also [@EnableWebMvc](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/servlet/config/annotation/EnableWebMvc.html) for quick examples.



### 如何回调WebMvcConfigurerAdapter？

在配置类中使用@EnableWebMvc，将从DelegatingWebMvcConfiguration进行重要配置。

```java
@Import(DelegatingWebMvcConfiguration.class)
  public @interface EnableWebMvc {
}
```

DelegatingWebMvcConfiguration是WebMvcConfigurationSupport的子类（它负责新基础架构的所有Spring配置）。配置类DelegatingWebMvcConfiguration具有以下依赖注入：

```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
   .....
  @Autowired(required = false)
  public void setConfigurers(List<WebMvcConfigurer> configurers) {
   	if (!CollectionUtils.isEmpty(configurers)) {
		this.configurers.addWebMvcConfigurers(configurers);
	}
  }
}
```

这意味着如果我们使用WebMvcConfigurerAdapter（WebMvcConfigurer的适配器实现）扩展我们的@Configuration类，那么它也会在上面注入。

在配置期间，在WebMvcConfigurationSupport中使用@Bean注释的各种工厂方法会导致对WebMvcConfigurer实现的回调。例如：

```java
public class WebMvcConfigurationSupport implements ... {
    @Bean
    public HandlerMapping viewControllerHandlerMapping() {
        ViewControllerRegistry registry = new ViewControllerRegistry();
        registry.setApplicationContext(this.applicationContext);
        addViewControllers(registry);
             ......
        return handlerMapping;
    }
    ....
    protected void addViewControllers(ViewControllerRegistry registry) {
    }
    ....
}
```

DelegatingWebMvcConfiguration重写了addViewControllers()方法，最终回调了我们的WebMvcConfigurerAdapter方法：

```java
 @Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
    ...
    @Override
	protected void addViewControllers(ViewControllerRegistry registry) {
	   this.configurers.addViewControllers(registry);
	}
    ...
}
```

configurers是相同的注入列表，正如我们在上面的代码片段中看到的那样。我们的@Configuration类看起来像这样：

```java
@EnableWebMvc
@Configuration
public class MyWebConfig extends WebMvcConfigurerAdapter {
  .....
   @Override
    public void addViewControllers (ViewControllerRegistry registry) {
        //our customization
    }
  ...
 }
```

## SpringServletContainerInitializer

SpringServletContainerInitializer类实现了 [ServletContainerInitializer](https://www.logicbig.com/tutorials/java-ee-tutorial/java-servlet/servlet-container-initializer-example.html)接口。这意味着这个类将被加载，并且在Servlet容器（版本3.0+）启动期间将调用其onStartup()方法，因为类路径上存在spring-web模块JAR。

### WebApplicationInitializer

SpringServletContainerInitializer被@HandlesTypes(WebApplicationInitializer.class)注解标注。这意味着，它的onStartup()方法与实现WebApplicationInitializer的所有类（在类路径上）一起传递。Spring将初始化所有这些具体类，并将调用其WebApplicationInitializer#onStartup(servletContext)方法。这些类可以在onStartup()方法中自由地进行任何编程servlet组件注册和初始化。

### Spring抽象WebApplicationInitializer实现

Spring还提供了此接口的抽象实现：AbstractDispatcherServletInitializer和AbstractAnnotationConfigDispatcherServletInitializer，它们已经注册了DispatcherServlet。其中一个抽象类的客户端实现可以进一步自定义注册过程。



AbstractAnnotationConfigDispatcherServletInitializer还初始化AnnotationConfigWebApplicationContext。客户端代码需要提供客户端配置类。以下代码段显示了如何实现其抽象方法：

```java
public class AppInitializer extends
          AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses () {
        return new Class<?>[]{MyAppConfig.class};
    }

    @Override
    protected Class<?>[] getServletConfigClasses () {
        return null;
    }

    @Override
    protected String[] getServletMappings () {
        return new String[]{"/"};
    }

```

## 使用Spring 5.0 WebMvcConfigurer默认方法

从Spring 5.0开始，WebMvcConfigurer具有Java 8默认方法。这意味着，对于MVC配置，我们可以直接实现此接口，而无需扩展WebMvcConfigurerAdapter（在5.0中已弃用）。

示例：实现WebMvcConfigurer接口

```java
@EnableWebMvc
@Configuration
@ComponentScan
public class MyWebConfig implements WebMvcConfigurer {

  @Override
  public void configureViewResolvers(ViewResolverRegistry registry) {
      registry.jsp("/WEB-INF/views/", ".jsp");
  }

  @Override
  public void addViewControllers(ViewControllerRegistry registry) {
      //this will map uri to jsp view directly without a controller
      registry.addViewController("/hi")
              .setViewName("hello");
  }
}
```

## 什么是DispatcherServlet.propertie?

DispatcherServlet.properties包含DispatcherServlet使用的默认策略对象/处理器列表。以下是5.1.8的内容：

```properties
# Default implementation classes for DispatcherServlet's strategy interfaces.
# Used as fallback when no matching beans are found in the DispatcherServlet context.
# Not meant to be customized by application developers.

org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver

org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver

org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping

org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
	org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter

org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver,\
	org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
	org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator

org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver

org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager
```

### 如何使用DispatcherServlet.properties？

DispatcherServlet在类加载期间加载这些属性。

在初始化不同策略（DispatcherServlet#initStrategies()）期间，DispatcherServlet会逐个检查这些策略是否已经注册；如果没有，它使用DispatcherServlet.properties设置的策略。这意味着如果我们在应用程序中注册不同的策略BEAN（如HandlerMapping, HandlerAdapter等）或者通过使用@EnableWebMvc等其他方式，不会使用DispatcherServlet.properties中列出的策略。