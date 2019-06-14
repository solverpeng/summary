Spring Boot Web是以 `WebMvcAutoConfiguration`作为入口进行配置的。几乎所有与Web相关的配置都在这个自动配置类中进行了说明。



# 静态资源

Spring Boot 有两种处理静态资源的方式。一种是第三方静态资源文件如何映射，另一种是我们自己的静态资源包改如何映射。Spring Boot 在 `WebMvcAutoConfiguration` 给出了说明，如下：

```java
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    // 是否支持默认的资源映射，默认为ture，支持
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
    } else {
        // 静态资源缓存设置
        Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
        CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
        // /webjars/** 请求处理
        if (!registry.hasMappingForPattern("/webjars/**")) {    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
        }
		
        // 处理静态资源路径：staticPathPattern = "/**"
        String staticPathPattern = this.mvcProperties.getStaticPathPattern();
        if (!registry.hasMappingForPattern(staticPathPattern)) {
            this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
        }

    }
}
```

```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {
    private static final String[] CLASSPATH_RESOURCE_LOCATIONS = {
			"classpath:/META-INF/resources/", "classpath:/resources/",
			"classpath:/static/", "classpath:/public/" };

	/**
	 * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
	 * /resources/, /static/, /public/].
	 */
	private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
}
```

## 第三方静态资源文件映射

```java
if (!registry.hasMappingForPattern("/webjars/**")) {    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
        }
```

可以看到对于所有的 `/webjars/**`的请求，都会去类路径 `classpath:/META-INF/resources/webjars/` 目录下查找对应的资源。



在 [webjars](https://www.webjars.org/) 中提供了绝大部分第三方静态资源各种版本的依赖（包含maven版本）。

![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/github/java-summary/springboot/springboot-webjars.png)



### 示例

以引入`jquery`为例，从`webjars`官网找到合适的`jquery`版本，选择对应的`maven`依赖导入到项目中。

![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/github/java-summary/springboot/webjars-jquery.png)

包结构如下：

![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/github/java-summary/springboot/springboot-jquery.png)

启动项目后，按照规则访问如下地址：http://localhost:8080/webjars/jquery/3.4.0/jquery.js。



## 程序中静态资源文件映射

对于我们自己编写的静态资源文件，该如何映射呢？

```java
String staticPathPattern = this.mvcProperties.getStaticPathPattern();
if (!registry.hasMappingForPattern(staticPathPattern)) {
    customizeResourceHandlerRegistration(
        registry.addResourceHandler(staticPathPattern)
        .addResourceLocations(getResourceLocations(
            this.resourceProperties.getStaticLocations()))
        .setCachePeriod(getSeconds(cachePeriod))
        .setCacheControl(cacheControl));
}
```

获取到的 `staticPathPattern` 默认是 `/**`。对于任意请求，在没有被其他`HandlerMapping`处理的情况下，都会到静态资源文件中进行查找。

从`this.resourceProperties.getStaticLocations()`可以获取到静态位置。包含如下位置：

> "classpath:/META-INF/resources/", "classpath:/resources/",
> "classpath:/static/", "classpath:/public/"

### 示例

![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/github/java-summary/springboot/spring-boot-static-img.png)

在类路径下添加了一个`img`文件夹，在里面添加了一张`timg.jpg`文件。

启动项目后，按规则进行访问：<http://localhost:8080/img/timg.jpg>



## 欢迎页

还是在`WebMvcAutoConfiguration`类中。

```java
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(
    ApplicationContext applicationContext) {
    return new WelcomePageHandlerMapping(
        new TemplateAvailabilityProviders(applicationContext),
        applicationContext, getWelcomePage(),
        this.mvcProperties.getStaticPathPattern());
}

private Optional<Resource> getWelcomePage() {
    String[] locations = getResourceLocations(
        this.resourceProperties.getStaticLocations());
    return Arrays.stream(locations).map(this::getIndexHtml)
        .filter(this::isReadable).findFirst();
}

private Resource getIndexHtml(String location) {
    return this.resourceLoader.getResource(location + "index.html");
}
```

可以看到是在静态资源文件夹下查找 `index.html`，作为欢迎页。

### 示例

在 `classpath:/resources/static/`下添加`index.html`文件。

启动项目后，访问：<http://localhost:8080/> 即可访问到首页信息。



## Favicon

`WebMvcAutoConfiguration`中有一个静态内部类 `FaviconConfiguration` 专门用来处理 Favicon。

```java
@Configuration
@ConditionalOnProperty(value = "spring.mvc.favicon.enabled",
                       matchIfMissing = true)
public static class FaviconConfiguration implements ResourceLoaderAware {

    private final ResourceProperties resourceProperties;

    private ResourceLoader resourceLoader;

    public FaviconConfiguration(ResourceProperties resourceProperties) {
        this.resourceProperties = resourceProperties;
    }

    @Override
    public void setResourceLoader(ResourceLoader resourceLoader) {
        this.resourceLoader = resourceLoader;
    }

    @Bean
    public SimpleUrlHandlerMapping faviconHandlerMapping() {
        SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
        mapping.setOrder(Ordered.HIGHEST_PRECEDENCE + 1);
        mapping.setUrlMap(Collections.singletonMap("**/favicon.ico",
                                                   faviconRequestHandler()));
        return mapping;
    }

    @Bean
    public ResourceHttpRequestHandler faviconRequestHandler() {
        ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
        requestHandler.setLocations(resolveFaviconLocations());
        return requestHandler;
    }

    private List<Resource> resolveFaviconLocations() {
        String[] staticLocations = getResourceLocations(
            this.resourceProperties.getStaticLocations());
        List<Resource> locations = new ArrayList<>(staticLocations.length + 1);
        Arrays.stream(staticLocations).map(this.resourceLoader::getResource)
            .forEach(locations::add);
        locations.add(new ClassPathResource("/"));
        return Collections.unmodifiableList(locations);
    }

}
```

可以看到，对于所有`**/favicon.ico`的请求，最终会在项目的静态资源目录下，以及类路径的的根目录`/`下查找对应的`favicon.ico`文件。如下：

静态资源目录下：

![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/github/java-summary/springboot/springboot-favicon2.png)

类路径的根目录：

![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/github/java-summary/springboot/springboot-favicon.png)

