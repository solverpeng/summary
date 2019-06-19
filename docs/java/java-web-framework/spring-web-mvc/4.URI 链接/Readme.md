# URI链接

本节介绍Spring Framework中可用于处理URI的各种选项。

## UriComponents

通过UriComponentsBuilder可以从URI Template构建URI，如以下示例所示：

```java
UriComponents uriComponents = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")  //①
        .queryParam("q", "{q}")  //②
        .encode() // ③
        .build(); // ④

URI uri = uriComponents.expand("Westin", "123").toUri();  //⑤
```

说明：

①：带有URI Template的静态工厂方法。

②：添加或替换URI组件。

③：请求编码URI Template和URI变量。

④：构建UriComponents

⑤：扩展变量并获取URI。

前面的示例可以进行合并，使用buildAndExpand缩短，如以下示例所示：

```java
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")
        .queryParam("q", "{q}")
        .encode()
        .buildAndExpand("Westin", "123")
        .toUri();
```

也可以直接转到URI，意味着编码进一步缩短它，如下例所示：

```java
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")
        .queryParam("q", "{q}")
        .build("Westin", "123");
```

使用完整的URI模板进一步缩短，如下例所示：

```java
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}?q={q}")
        .build("Westin", "123");
```



## UriBuilder

UriComponentsBuilder实现了UriBuilder。可以使用UriBuilderFactory创建一个UriBuilder。UriBuilderFactory和UriBuilder一起提供了一种可插入的机制，可以根据共享配置（例如基本URL，编码首选项和其他详细信息）从URI Template构建URI。



可以使用UriBuilderFactory配置RestTemplate和WebClient以自定义URI。

DefaultUriBuilderFactory是UriBuilderFactory的默认实现，它在内部使用UriComponentsBuilder并公开共享配置选项。



以下示例显示如何配置RestTemplate：

```java
// import org.springframework.web.util.DefaultUriBuilderFactory.EncodingMode;

String baseUrl = "https://example.org";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl);
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VARIABLES);

RestTemplate restTemplate = new RestTemplate();
restTemplate.setUriTemplateHandler(factory);
```

以下示例配置WebClient：

```java
// import org.springframework.web.util.DefaultUriBuilderFactory.EncodingMode;

String baseUrl = "https://example.org";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl);
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VARIABLES);

WebClient client = WebClient.builder().uriBuilderFactory(factory).build();
```

此外，还可以直接使用DefaultUriBuilderFactory。它类似于使用UriComponentsBuilder，但它不是静态工厂方法，而是一个保存配置和首选项的实际实例，如下例所示：

```java
String baseUrl = "https://example.com";
DefaultUriBuilderFactory uriBuilderFactory = new DefaultUriBuilderFactory(baseUrl);

URI uri = uriBuilderFactory.uriString("/hotels/{hotel}")
        .queryParam("q", "{q}")
        .build("Westin", "123");
```

## URI Encoding

UriComponentsBuilder在两个级别公开编码选项：

- UriComponentsBuilder#coding()：首先对URI模板进行预编码，然后在扩展时严格编码URI变量。
- UriComponents #coding()：扩展URI变量后对URI组件进行编码。

这两个选项都使用转义的八位字节替换非ASCII和非法字符。但是，第一个选项还会替换出现在URI变量中的保留含义的字符。

> 考虑“;”，这在路径中是合法的但具有保留意义。第一个选项取代“;”在URI变量中使用“％3B”但在URI模板中没有。相比之下，第二个选项永远不会替换“;”，因为它是路径中的合法字符。

对于大多数情况，第一个选项可能会给出预期结果，因为它将URI变量视为完全编码的不透明数据，而选项2仅在URI变量故意包含保留字符时才有用。

以下示例使用第一个选项：

```java
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}")
            .queryParam("q", "{q}")
            .encode()
            .buildAndExpand("New York", "foo+bar")
            .toUri();

    // Result is "/hotel%20list/New%20York?q=foo%2Bbar"
```

可以通过直接转到URI（这意味着编码）来缩短前面的示例，如下例所示：

```java
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}")
            .queryParam("q", "{q}")
            .build("New York", "foo+bar")
```

可以使用完整的URI模板进一步缩短它，如下例所示：

```java
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}?q={q}")
            .build("New York", "foo+bar")
```

WebClient和RestTemplate通过UriBuilderFactory策略在内部扩展和编码URI Template。两者都可以配置自定义策略。如下例所示：

```java
String baseUrl = "https://example.com";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl)
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VALUES);

// Customize the RestTemplate..
RestTemplate restTemplate = new RestTemplate();
restTemplate.setUriTemplateHandler(factory);

// Customize the WebClient..
WebClient client = WebClient.builder().uriBuilderFactory(factory).build();
```

DefaultUriBuilderFactory实现在内部使用UriComponentsBuilder来扩展和编码URI Template。作为工厂，它提供了一个单独的位置来配置编码方法，基于以下编码模式之一：

- TEMPLATE_AND_VALUES：使用UriComponentsBuilder#encode()，对应于前面列表中的第一个选项，对URI模板进行预编码，并在扩展时严格编码URI变量。
- VALUES_ONLY：不对URI模板进行编码，而是在将URI变量扩展到模板之前，通过UriUtils #creditUriUriVariables将严格编码应用于URI变量。
- URI_COMPONENTS：使用UriComponent#encode()（对应于前面列表中的第二个选项），在URI变量扩展后对URI组件值进行编码。
- NONE：不应用编码。

出于历史原因和向后兼容性，RestTemplate设置为EncodingMode.URI_COMPONENTS。WebClient依赖于DefaultUriBuilderFactory中的默认值，该值已从5.0.x中的EncodingMode.URI_COMPONENTS更改为5.1中的EncodingMode.TEMPLATE_AND_VALUES。



## Relative Servlet Requests

可以使用ServletUriComponentsBuilder创建相对于当前请求的URI，如以下示例所示：

```java
HttpServletRequest request = ...

// Re-uses host, scheme, port, path and query string...

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromRequest(request)
        .replaceQueryParam("accountId", "{id}").build()
        .expand("123")
        .encode();
```

您可以创建相对于上下文路径的URI，如以下示例所示：

```java
// Re-uses host, port and context path...

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromContextPath(request)
        .path("/accounts").build()
```

您可以创建相对于Servlet的URI（例如，/main/*），如以下示例所示：

```java
// Re-uses host, port, context path, and Servlet prefix...

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromServletMapping(request)
        .path("/accounts").build()
```

> 从5.1开始，ServletUriComponentsBuilder忽略来自Forwarded和X-Forwarded- *标头的信息，这些标头指定了客户端发起的地址。考虑使用ForwardedHeaderFilter来提取和使用或丢弃此类标头。



## Links to Controllers

Spring MVC提供了一种准备控制器方法链接的机制。例如，以下MVC控制器允许创建链接：

```java
@Controller
@RequestMapping("/hotels/{hotel}")
public class BookingController {

    @GetMapping("/bookings/{booking}")
    public ModelAndView getBooking(@PathVariable Long booking) {
        // ...
    }
}
```

可以通过按名称引用方法来准备链接，如以下示例所示：

```java
UriComponents uriComponents = MvcUriComponentsBuilder
    .fromMethodName(BookingController.class, "getBooking", 21).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```

在前面的示例中，我们提供了实际的方法参数值（在本例中，long值：21），用作路径变量并插入到URL中。此外，我们提供值42来填充任何剩余的URI变量，例如从类型级请求映射继承的hotel变量。如果方法有更多参数，我们可以为URL不需要的参数提供null。通常，只有@PathVariable和@RequestParam参数与构造URL相关。



还有其他方法可以使用MvcUriComponentsBuilder。例如，可以使用类似于通过代理进行模拟测试的技术，以避免按名称引用控制器方法，如以下示例所示（该示例假定MvcUriComponentsBuilder.on的静态导入）：

```java
UriComponents uriComponents = MvcUriComponentsBuilder
    .fromMethodCall(on(BookingController.class).getBooking(21)).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```

> 当控制器方法签名可用于与fromMethodCall创建链接时，它们的设计受到限制。除了需要正确的参数签名之外，返回类型还存在技术限制（即，为链接构建器调用生成运行时代理），因此返回类型不能是final。特别是，视图名称的常见String返回类型在此处不起作用。您应该使用ModelAndView甚至是普通的Object（带有String返回值）。

前面的示例在MvcUriComponentsBuilder中使用静态方法。在内部，它们依赖ServletUriComponentsBuilder从当前请求的方案，主机，端口，上下文路径和servlet路径准备基本URL。这在大多数情况下效果很好。但是，有时，它可能是不够的。例如，您可能在请求的上下文之外（例如准备链接的批处理）或者您可能需要插入路径前缀（例如从请求路径中删除并需要重新插入链接的区域设置前缀）。



对于这种情况，您可以使用接受UriComponentsBuilder的静态fromXxx重载方法来使用基本URL。或者，您可以使用基本URL创建MvcUriComponentsBuilder的实例，然后使用基于实例的withXxx方法。例如，以下列表使用withMethodCall：

```java
UriComponentsBuilder base = ServletUriComponentsBuilder.fromCurrentContextPath().path("/en");
MvcUriComponentsBuilder builder = MvcUriComponentsBuilder.relativeTo(base);
builder.withMethodCall(on(BookingController.class).getBooking(21)).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```

从5.1开始，MvcUriComponentsBuilder忽略来自`Forwarded`和`X-Forwarded- *`标头的信息，它指定客户端发起的地址。考虑使用ForwardedHeaderFilter来提取和使用或丢弃此类标头。



## Links in Views

在Thymeleaf，FreeMarker或JSP等视图中，可以通过引用每个请求映射的隐式或显式指定名称来构建指向带注释控制器的链接。

请考虑以下示例：

```java
@RequestMapping("/people/{id}/addresses")
public class PersonAddressController {

    @RequestMapping("/{country}")
    public HttpEntity getAddress(@PathVariable String country) { ... }
}
```

给定前面的控制器，您可以从JSP准备链接，如下所示：

```java
<%@ taglib uri="http://www.springframework.org/tags" prefix="s" %>
...
<a href="${s:mvcUrl('PAC#getAddress').arg(0,'US').buildAndExpand('123')}">Get Address</a>
```

前面的示例依赖于Spring标记库（即META-INF/spring.tld）中声明的mvcUrl函数，但很容易定义自己的函数或为其他模板技术准备类似的函数。

这是如何工作的。在启动时，每个@RequestMapping都通过HandlerMethodMappingNamingStrategy分配一个默认名称，其默认实现使用类的大写字母和方法名称（例如，ThingController中的getThing方法变为“TC#getThing”）。如果存在名称冲突，则可以使用@RequestMapping(name =“..”)分配显式名称或实现自己的HandlerMethodMappingNamingStrategy。