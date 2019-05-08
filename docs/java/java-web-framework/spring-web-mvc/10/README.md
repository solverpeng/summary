# Spring MVC测试

Spring MVC有如下测试的办法

- Servlet API Mocks：模拟Servlet API的实现，用于单元测试Contrllers，Filters和其他Web组件。
- TestContent 框架：支持在`JUnit`和`TestNG`测试中加载Spring配置，包括跨测试方法高效缓存已加载的配置，以及支持使用`MockServletContext`加载`WebApplicationContext`。
- Spring MVC Test：一个框架，也称为`MockMvc`，用于通过`DispatcherServlet`测试带注解的控制器，包含全部的Spring MVC基础结构但没有HTTP服务器。
- Client-side REST：`spring-test`提供了一个`MockRestServiceServer`，可以将其用作模拟服务器，用于测试内部使用`RestTemplate`的客户端代码。
- WebTestClient：专为测试`WebFlux`应用程序而构建，但它也可用于通过HTTP连接对任何服务器进行端到端集成测试。是一个非阻塞，被动的客户端，非常适合测试异步和流式方案。



## Spring MVC Test

`Spring MVC test`框架为测试 Spring MVC代码提供了一流的支持。

它构建在`spring-test`模块的Servlet API mock 对象上，不使用正在运行的servlet容器。使用`DispatcherServlet`提供完整的SpringMVC运行时行为，并支持使用`TextContext`框架加载实际的Spring配置。另外还有一种模式，可以手动实例化控制器，然后一次测试一个。

`Spring MVC Test`框架还为使用`RestTemplate`的测试代码提供客户端支持。客户端测试mock服务器响应，不使用正在运行的服务器。

### 服务器端测试

可以使用JUnit或TestNG为Spring MVC控制器编写一个普通的单元测试。实例化一个控制器，使用mock注入这个控制器，然后调用其方法（根据需要传入`MockHttpServletRequest`或`MockHttpServletResponse`或其他）。然而，编写这样的单元测试时，还有许多内容未测试：如`reqeust mappings`、`data binging`、`type conversion`、`validation`等等。另外，在一个请求的生命周期中，还有许多控制器方法也可能会被调用，如`@InitBinder`、`@ModelAttribute`、`@ExceptionHandler`。

Spring MVC Test框架的目标是，执行请求并通过实际的`DispatcherServlet`，然后生成响应，来测试控制器的有效方法。

Spring MVC Test建立在`spring-test`模块中常见的`Servlet API mock`实现上。这就允许执行请求并生成响应，而无需在Servlet容器中运行。下面是一个例子：

```java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringJUnitWebConfig(locations = "test-servlet-context.xml")
class ExampleTests {

    private MockMvc mockMvc;

    @BeforeEach
    void setup(WebApplicationContext wac) {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    }

    @Test
    void getAccount() throws Exception {
        this.mockMvc.perform(get("/accounts/1")
                .accept(MediaType.parseMediaType("application/json;charset=UTF-8")))
                .andExpect(status().isOk())
                .andExpect(content().contentType("application/json"))
                .andExpect(jsonPath("$.name").value("Lee"));
    }
}
```

上面的例子依赖于`Test Context`框架的`WebApplicationContext`支持，以便加载与测试类相同的包中的Spring配置文件，同样也支持基于`Java`配置。

`MockMvc`实例用于对`/accounts/1`生成一个`GET`请求，并验证生成的响应状态码为200，content-type为`application/json`，且响应体有一个名为`name`，值为`Lee`的JSON属性。Jayway [JsonPath](https://github.com/json-path/JsonPath)项目支持`jsonPath`语法。

####  静态导入

上面的例子静态导入了一些包，如`MockMvcRequestBuilders.*`，`MockMvcResultMatchers.*`。一个简单的方式搜索这些类是，匹配`MockMvc*`。

#### 设置选择

有两种方式创建`MockMvc`实例。第一种是通过`Test Context`框架加载`Spring MVC`配置文件，该框架加载`Spring MVC`配置文件并将`WebApplicationContext`注入到测试类中，用于构建一个`MockMvc`实例。如下：

```java
@RunWith(SpringRunner.class)
@WebAppConfiguration
@ContextConfiguration("my-servlet-context.xml")
public class MockMvcChoice1 {
    @Autowired
    private WebApplicationContext wac;

    private MockMvc mockMvc;

    @Before
    public void setup() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    }

    @Test
    public void test() {
        DispatcherServlet dispatcherServlet = mockMvc.getDispatcherServlet();
        List<HandlerMapping> handlerMappings = dispatcherServlet.getHandlerMappings();
        assert handlerMappings != null;
        for (HandlerMapping handlerMapping : handlerMappings) {
            System.out.println(handlerMapping);
        }
    }
}
```

第二种方式是手动创建一个控制器实例而不需要加载Spring配置。默认会自动创建与`MVC`J Java 配置或`MVC`命名空间大致相同的配置，也可以自定义。如下：

```java
public class MockMvcChoice2 {
    private MockMvc mockMvc;

    @Before
    public void setup() {
        this.mockMvc = MockMvcBuilders.standaloneSetup(new PersonController()).build();
    }

    @Test
    public void test() {
        DispatcherServlet dispatcherServlet = mockMvc.getDispatcherServlet();
        List<HandlerMapping> handlerMappings = dispatcherServlet.getHandlerMappings();
        assert handlerMappings != null;
        for (HandlerMapping handlerMapping : handlerMappings) {
            System.out.println(handlerMapping);
        }
    }
}
```

该选用哪种方式呢？

第一种方式，`webAppContextSetup`加载了实际的`Spring MVC`配置，从而实现了更完整的集成测试。由于`TestContext`框架缓存了加载的`Spring`配置，因此即使引入了更多的测试，它也有助于保持测试快速运行。此外，您可以通过`Spring`配置将`mock`服务注入控制器，以便专注于测试Web层。

下面的例子使用`MOckito`声明`mock`服务：

```xml
<bean id="personService" class="org.mockito.Mockito" factory-method="mock">
    <constructor-arg value="com.solverpeng.service.PersonService"/>
</bean>
```

然后，可以将模拟服务注入测试，进而进行验证，如下：

```java
@RunWith(SpringRunner.class)
@WebAppConfiguration
@ContextConfiguration("test-servlet-context.xml")
public class PersonTests {
    @Autowired
    private WebApplicationContext wac;

    private MockMvc mockMvc;

    @Autowired
    private PersonService personService;

    // ...
}
```

第二种方式，`standaloneSetup`，更接近单元测试，一次测试一个控制器。可以使用mock依赖手动注入控制器，且不需要调用Spring配置文件。这些测试更侧重于样式，并且更容易看到正在测试哪个控制器，是否需要任何特定的Spring MVC配置，等等。`standaloneSetup`也是编写临时测试以验证特定行为或调试问题的一种非常方便的方法。

#### 特性设置

无论您使用哪个`MockMvc`构建器，所有`MockMvcBuilder`实现都提供了一些常用且非常有用的功能。例如：可以为所有请求声明一个`Accept`标头，并期望所有响应状态码为200以及Content-Type标头设置，如下所示：

```java
this.mockMvc = MockMvcBuilders.standaloneSetup(new PersonController()) .defaultRequest(MockMvcRequestBuilders.get("/").accept(MediaType.APPLICATION_JSON))
                .alwaysExpect(status().isOk())
                .alwaysExpect(content().contentType("application/json;charset=UTF-8"))
                .build();
```

此外，第三方框架（和应用程序）可以预打包设置指令，例如`MockMvcConfiger`中的指令。Spring框架有一个这样的内置实现，它有助于保存和跨请求重用HTTP会话。如下：

```java
MockMvc mockMvc = MockMvcBuilders.standaloneSetup(new TestController())
        .apply(sharedHttpSession())
        .build();
```

#### 执行请求

可以使用任何HTTP方法来执行请求。如下：

```java
mockMvc.perform(post("/hotels/{id}", 42).accept(MediaType.APPLICATION_JSON));
```

还可以执行内部使用`MockMultipartHttpServletRequest`的文件上传请求，这样就不会实际解析多部分请求。必须将其设置为类似于以下示例：

```java
mockMvc.perform(multipart("/doc").file("a1", "ABC".getBytes("UTF-8")));
```

也可以在URI模板样式中指定查询参数，如下例所示：

```java
mockMvc.perform(get("/hotels?thing={thing}", "somewhere"));
```

还可以添加表示查询或表单参数的Servlet请求参数：

```java
mockMvc.perform(get("/hotels").param("thing", "somewhere"));
```

在大多数情况下，最好将上下文路径和servlet路径保留在请求URI之外。若必须使用完整请求URI进行测试，则需要确保设置了`contextPath`和`servletPath`，以便请求映射有效。如下：

```java
mockMvc.perform(get("/app/main/hotels/{id}").contextPath("/app").servletPath("/main"))
```

为每个请求设置`contextPath`和`servletPath`是很麻烦的。可以设置默认的请求属性来替代它。如下：

```java
MockMvc mockMvc = standaloneSetup(new AccountController())
            .defaultRequest(get("/")
            .contextPath("/app").servletPath("/main")
            .accept(MediaType.APPLICATION_JSON)).build();
```

前面的例子是全局配置，通过`MockMvc`实例执行的每个请求都会受到影响。也可以在具体的某个请求中覆盖上述设置。

#### 定义期望结果

您可以通过在执行请求后附加一个或多个`.andExpect(..)`调用来定义期望结果。如下：

```java
mockMvc.perform(get("/accounts/1")).andExpect(status().isOk());
```

`MockMvcResultMatchers.*`提供了一系列期望值。期望结果分为两大类，第一类是断言验证响应的属性（如响应状态、标头、内容）。另一类是断言超出响应范围。这些断言让你检查`Spring MVC`的特定方面，比如哪个控制器方法处理了请求，是否引发和处理异常，模型的内容是什么，选择了哪个视图，添加了哪些Flash属性等等。还允许您检查Servlet特定方面，例如请求和会话属性。

以下测试断言绑定或验证失败：

```java
mockMvc.perform(post("/persons"))
    .andExpect(status().isOk())
    .andExpect(model().attributeHasErrors("person"));
```

很多时候，在编写测试时，打印执行的请求的结果很有用。如下：

```java
mockMvc.perform(post("/persons"))
    .andDo(print())
    .andExpect(status().isOk())
    .andExpect(model().attributeHasErrors("person"));
```

只要请求处理不会导致未处理的异常，`print()`方法就会将所有可用的结果数据打印到`System.out`。如果希望记录结果数据而不是打印结果数据，可以调用`log()`方法，该方法将结果数据作为单个调试消息记录在`org.springframework.test.web.servlet.result logging`类别下。

在某些情况下，您可能希望直接访问结果，并验证其他情况下无法验证的内容。这可以通过在所有其他期望之后附加`.andReturn()`来实现，如下示例所示：

```java
MvcResult mvcResult = mockMvc.perform(post("/persons")).andExpect(status().isOk()).andReturn();
```

如果所有测试重复相同的期望，您可以在构建MockMvc实例时设置一次共同期望，如以下示例所示：

```java
standaloneSetup(new SimpleController())
    .alwaysExpect(status().isOk())
    .alwaysExpect(content().contentType("application/json;charset=UTF-8"))
    .build()
```

当一个JSON响应包含一个超文本链接时，可以通过`Json Path`表达式进行验证。如下：

```java
mockMvc.perform(get("/people").accept(MediaType.APPLICATION_JSON))
    .andExpect(jsonPath("$.links[?(@.rel == 'self')].href").value("http://localhost:8080/people"));
```

当一个XML响应包含超文本链接时，可以使用 `XPath`表达式进行验证：

```java
Map<String, String> ns = Collections.singletonMap("ns", "http://www.w3.org/2005/Atom");
mockMvc.perform(get("/handle").accept(MediaType.APPLICATION_XML))
    .andExpect(xpath("/person/ns:link[@rel='self']/@href", ns).string("http://localhost:8080/people"));
```

#### 过滤器注册

当设置一个`MockMvc`后，可以注册一个或多个`Filter`实例。如下：

```java
mockMvc = standaloneSetup(new PersonController()).addFilters(new CharacterEncodingFilter()).build();
```

注册的过滤器通过`spring-test`中的`MockFilterChain`调用，最后一个过滤器委托给`DispatcherServlet`。










