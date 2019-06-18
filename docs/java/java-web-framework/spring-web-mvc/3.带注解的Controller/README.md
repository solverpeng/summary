# 带注解的Controller

Spring MVC提供了一个基于注解的编程模型，其中`@Controller`和`@RestController`组件使用注解表示请求映射，请求输入，异常处理等等。带注解的Controller具有灵活的方法签名，不必扩展基类，也不必实现特定的接口。例子如下：

```java
@Controller
public class HelloController {

    @GetMapping("/hello")
    public String handle(Model model) {
        model.addAttribute("message", "Hello World!");
        return "index";
    }
}
```

上面的例子接受一个`Model`并且返回一个字符串形式的视图名。

## 声明

在 Servlet 的`WebApplicationContext`中，你可以使用 Spring bean definition 定义一个控制器Bean。@Controller构造型允许自动检测，与Spring一致支持，以检测类路径中的@Component类并自动注册它们的bean定义。它还充当带注解的类的构造型，表明它作为Web组件的角色。



启用@Controller bean类的自动检测，可以在Java配置文件中添加组件扫描，如下：

```java
@Configuration
@ComponentScan("org.example.web")
public class WebConfig {

    // ...
}
```

也可以使用XML的方式：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example.web"/>

    <!-- ... -->

</beans>
```

@RestController是一个由@Controller和@ResponseBody元注解组成的组合注解，表示控制器内每个方法继承类型级别@ResponseBody注解，因此，它直接写入响应主体，而不是视图解析，并使用HTML模板呈现。



### AOP代理

在某些情况下，您可能需要在运行时用 AOP 代理来修饰控制器。一个例子是，如果您选择直接在控制器上使用@Transactional注释。在这种情况下，特别是对于控制器，我们建议使用基于类的代理。这通常是控制器的默认选择。但是，如果控制器必须实现一个不是Spring上下文回调的接口（如InitializingBean、*Aware和其他），则可能需要显式配置基于类的代理。例如，使用`<tx:annotationdriven/>`，您可以更改为`<tx:annotation-driven proxy-target-class="true"/>`。



## 请求映射

> **不论是何种方式的映射方式，目的都是为了缩小请求映射范围**

可以使用`@RequestMapping`注解将请求映射到控制器方法。它具有多种属性，可以通过URL、HTTP方法、请求参数、标头和媒体类型进行匹配。也可以在类级别使用它来表示共享映射，或者在方法级别使用它来缩小到特定端点映射的范围。

`@RequestMapping`还有一些特定的`HTTP`方法的快捷方式变体：

- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping`

快捷方式是提供的自定义注解，大多数控制器方法应该映射到特定的HTTP方法，而不是使用`@RequestMapping`，默认情况下，它与所有HTTP方法匹配。同样，在类级别仍然需要`@RequestMapping`来表示共享映射。下面的例子展示了类和方法级别的映射：

```java
@RestController
@RequestMapping("/persons")
class PersonController {

    @GetMapping("/{id}")
    public Person getPerson(@PathVariable Long id) {
        // ...
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public void add(@RequestBody Person person) {
        // ...
    }
}
```

### URI规则

1. 使用通配符映射请求
   - ? 匹配一个字符
   - `*` 匹配0个或多个字符在一个路径段中
   - `**` 匹配0个或多个路径段

2. 声明URI变量，通过`@PathVariable`访问

   - ```java
     @GetMapping("/owners/{ownerId}/pets/{petId}")
     public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
         // ...
     }
     ```

   - 也可以在类和方法级别声明URI变量

   - ```java
     @Controller
     @RequestMapping("/owners/{ownerId}")
     public class OwnerController {
     
         @GetMapping("/pets/{petId}")
         public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
             // ...
         }
     }
     ```

   - 也可以通过`@PathVariable`的`value`属性值来明确的指定URI变量。

   - URI变量默认会自动转换为合适的类型，或者会提示`TypeMismatchException`的异常。默认也支持基本数据类型，也可以注册其他数据类型。

3. 语法`{varName:regex}`声明一个URI变量，其正则表达式的语法为`{varName:regex}`。例如，给定URL`“/spring-web-3.0.5.jar”`，用以下方法提取名称，版本和文件扩展名：

   - ```java
     @GetMapping("/{name:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{ext:\\.[a-z]+}")
     public void handle(@PathVariable String version, @PathVariable String ext) {
         // ...
     }
     ```

4. URI路径规则也可以使用占位符从外部配置文件中读取，通过`PropertyPlaceHolderConfigurer`来解析外部文件。

Spring MVC 使用`PathMatcher`和Spring-core的`AntPathMatcher`实现进行URL路径匹配。

### 规则比较

当多个规则与URL匹配时，必须对它们进行比较以找到最佳匹配。这是通过使用`AntPathMatcher.getPatternComparator(String path)`来完成的。

### 后缀匹配

默认情况下，Spring MVC执行`.*`后缀模式匹配，映射到`/person`的控制器也隐式映射到`/person.*`。使用文件扩展名来代替用于响应的请求内容类型（即，代替Accept标头）。如`/person.pdf`, `/person.xml`等。

当浏览器用于发送难以一致的Accept标头时，必须以这种方式使用文件扩展名。目前，使用Accept头应该是首选。随着时间的推移，文件扩展名的使用已经证明有多种方式存在问题。当使用URI变量，路径参数和URI编码进行覆盖时，它可能会导致歧义。关于基于URL的授权和安全性的推理也变得更加困难。

要完全禁用文件扩展名使用如下方式：

- `useSuffixPatternMatching(false)`
- `favorPathExtension(false)`

基于URL的内容协议仍然有用（例如，在浏览器中键入URL时）。为了实现这一点，我们建议使用基于查询参数的策略，以避免文件扩展名带来的大多数问题。或者，如果必须使用文件扩展名，请考虑通过`ContentNegotiationConfigurer`的`MediaTypes`属性将它们限制为显式注册的扩展名列表。

### 可消费的媒体类型

可以根据请求的`Content-Type`缩小请求映射，必须指定请求的`Content-Type`，否则不能映射，如下：

```java
@PostMapping(path = "/pets", consumes = "application/json") 
public void addPet(@RequestBody Pet pet) {
    // ...
}
```

使用`consumes`属性来缩小内容类型的映射范围。还支持否定表达式，如`!text/plain`表示除`text/plain`之外的任何内容类型。

可以在类级别声明共享使用该属性。方法级别的使用该属性会覆盖类级别的声明。

`MediaType`为常用媒体类型提供常量，例如`APPLICATION_JSON_VALUE`和`APPLICATION_XML_VALUE`。

### 可生产的媒体类型

可以根据`Accept`请求头和控制器方法生成的内容类型列表来缩小请求映射，不是必须指定请求的`Accept`，如不指定，也会根据`produces`值进行映射，如下：

```java
@GetMapping(path = "/pets/{petId}", produces = "application/json;charset=UTF-8") 
@ResponseBody
public Pet getPet(@PathVariable String petId) {
    // ...
}
```

使用`produce`属性来缩小内容类型的映射。

媒体类型可以指定字符集。也支持否定表达式。

对于JSON内容类型，即使`RFC7159`明确指出“不需要定义charset参数”，也应指定UTF-8字符集，因为某些浏览器要求它正确解释UTF-8特殊字符。

同样也可以在类级别声明使用该属性。方法级别的使用该属性会覆盖类级别的声明。

### 请求参数和标头

可以根据请求参数条件缩小请求映射范围。测试是否存在请求参数（`myParam`），不存在（`!myParam`）或者特定值（`myParam=myValue`）。

```java
@GetMapping(path = "/pets/{petId}", params = "myParam=myValue") 
public void findPet(@PathVariable String petId) {
    // ...
}
```

测试`myParam`是否等于`myValue`。

标头也与请求参数类似。

```java
@GetMapping(path = "/pets", headers = "myHeader=myValue") 
public void findPet(@PathVariable String petId) {
    // ...
}
```

测试`myHeader`标头是否等于`myValue`。

可以将`Content-Type`和`Accept`与`headers`条件匹配，但最好使用consume和`produces`替代。

### 自定义注解

Spring MVC还支持 使用自定义请求匹配逻辑的 自定义请求映射属性。这是一个更高级的选项，需要继承`RequestMappingHandlerMapping`并覆盖`getCustomMethodCondition`方法，您可以在其中检查自定义属性并返回自己的`RequestCondition`。

### 明确的注册

可以以编程方式注册处理程序方法，您可以将其用于动态注册或高级情况，例如不同URL下的同一处理程序的不同实例，如下：

```java
@Configuration
public class MyConfig {

    @Autowired
    public void setHandlerMapping(RequestMappingHandlerMapping mapping, UserHandler handler) 
            throws NoSuchMethodException {

        RequestMappingInfo info = RequestMappingInfo
                .paths("/user/{id}").methods(RequestMethod.GET).build(); 

        Method method = UserHandler.class.getMethod("getUser", Long.class); 

        mapping.registerMapping(info, handler, method); 
    }

}
```

## 处理方法

### 方法参数

下表描述了控制器方法支持的参数。

| 控制器方法参数                                               | 描述                                                         |
| ------------------------------------------------------------ | :----------------------------------------------------------- |
| `WebRequest`, `NativeWebRequest`                             | 不使用Servlet API，即可直接访问请求参数，Request和Session属性 |
| `javax.servlet.ServletRequest`, `javax.servlet.ServletResponse` | 选择任意特定的请求或响应类型。<br />如：`ServletRequest`, `HttpServletRequest`；或者是Spring的 `MultipartRequest`, `MultipartHttpServletRequest` |
| `javax.servlet.http.HttpSession`                             | Session会话                                                  |
| `javax.servlet.http.PushBuilder`                             |                                                              |
| `java.security.Principal`                                    | 当前经过认证的用户。可能是特定`Principal`的实现类            |
| `HttpMethod`                                                 | 当前请求的请求方法                                           |
| `java.util.Locale`                                           | 当前请求区域。由可用的`LocaleResolver`指定                   |
| `java.util.TimeZone` + `java.time.ZoneId`                    | 当前请求关联的时区，由`LocaleContextResolver`指定            |
| `java.io.InputStream`, `java.io.Reader`                      | 用于访问Servlet API的原始请求主体                            |
| `java.io.OutputStream`, `java.io.Writer`                     | 用于访问Servlet API的原始响应主体                            |
| `@PathVariable`                                              | 用于访问URI路径模板变量                                      |
| `@MatrixVariable`                                            | 用于访问URI路径段中的 `name-value` 对                        |
| @RequestParam                                                | 用于访问Servlet请求参数，包括 multipart 文件。<br />参数值将转换为声明的方法参数类型。<br />对于简单的参数值，该参数不是必须的 |
| @RequestHeader                                               | 用于访问请求头。<br />标头值将转换为声明的方法参数类型       |
| @CookieValue                                                 | 用于访问Cookie。<br />Cookie值将转换为声明的方法参数类型     |
| @RequestBody                                                 | 用于访问HTTP请求体。<br />通过使用 HttpMessageConverter 实现将请求体内容转换为声明的方法参数类型 |
| `HttpEntity<B>`                                              | 用于访问HTTP请求头和请求体。<br />使用HttpMessageConverter转换请求体 |
| @RequestPart                                                 | 用于访问`multipart/form-data`请求中的部件。<br />使用HttpMessageConverter转换部件 |
| `java.util.Map`, `org.springframework.ui.Model`, `org.springframework.ui.ModelMap` | 用于访问HTML控制器中使用的模型，并将其作为视图呈现的一部分暴露给模板 |
| RedirectAttributes                                           | 指定重定向的属性                                             |
| @ModelAttribute                                              | 用于访问模型中的现有属性（如果不存在则实例化），并且应用于数据绑定和验证。<br />@ModelAttribute 是可选值。 |
| `Errors`, `BindingResult`                                    | 用于访问来自命令对象的验证和数据绑定的错误（即@ModelAttribute参数）或来自验证@RequestBody或@RequestPart参数的错误。<br />必须在经过验证的方法参数之后立即声明Errors或BindingResult参数。 |
| `SessionStatus` + class-level `@SessionAttributes`           | 用于标记表单处理完成，它触发通过类级别@SessionAttributes注释声明的会话属性的清除 |
| UriComponentsBuilder                                         | 用于准备相对于当前请求的 host, port, scheme，上下文路径和servlet映射的文字部分的URL |
| @SessionAttribute                                            | 用于访问任何会话属性，与由于类级别@SessionAttributes声明而存储在会话中的模型属性相反 |
| @RequestAttribute                                            | 用于访问请求属性                                             |
| 任意的其他参数                                               | 如果方法参数与此表中前面值不匹配，并且它是一个简单类型（由BeanUtils#isSimpleProperty确定），则它被解析为@RequestParam。否则，它将被解析为@ModelAttribute。 |

### 返回值

下表描述了控制器方法支持的返回值。

| 控制器方法返回值                                             | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| @ResponseBody                                                | 通过HttpMessageConverter实现将返回值转换并写入响应。         |
| `HttpEntity<B>`, `ResponseEntity<B>`                         | 通过HttpMessageConverter实现将指定的完整响应（包括HTTP头和主体）返回值转换并写入响应。 |
| HttpHeaders                                                  | 用于返回带有响应头但没有相应体的响应                         |
| String                                                       | 返回一个使用`ViewResolver`实现解析的视图名，并与隐式的模型（通过命令对象和@ModelAttribute确定）一起使用。<br />处理器方法还可以通过声明Model参数以编程方式丰富模型。 |
| View                                                         | 用于与隐式模型（通过命令对象和@ModelAttribute方法确定）一起呈现的View实例。<br />处理器方法还可以通过声明Model参数以编程方式丰富模型。 |
| `java.util.Map`, `org.springframework.ui.Model`              | 要添加到隐式模型的属性，通过`RequestToViewNameTranslator`隐式的确定视图名称。 |
| @ModelAttribute                                              | 要添加到模型的属性，通过`RequestToViewNameTranslator`隐式确定视图名称。是可选的。 |
| `ModelAndView` 对象                                          | 返回要使用的视图和模型，以及可选的响应状态。                 |
| void                                                         | 如果具有void返回类型（或返回值null）的方法，且它还具有ServletResponse，OutputStream参数或@ResponseStatus注解，则认为已完全处理该响应。<br />如果控制器已经进行了 ETag 或 lastModified 时间戳检查，情况也是如此。<br />如果不是以上的情况，则void返回类型也可以表示REST控制器的“无响应主体”或HTML控制器选择默认视图名称。 |
| `DeferredResult<V>`                                          | 从任何线程异步生成任何前面的返回值。例如，由于某些事件或回调。 |
| `ListenableFuture<V>`, <br/>`java.util.concurrent.CompletionStage<V>`, <br/>`java.util.concurrent.CompletableFuture<V>` | 作为方便，DeferredResult的替代方案                           |
| `ResponseBodyEmitter`, <br/>`SseEmitter`                     | 使用HttpMessageConverter实现以异步方式发送对象流以写入响应。 |
| StreamingResponseBody                                        | 异步写入响应OutputStream。也支持ResponseEntity的主体。       |
| 任意其他类型                                                 | 任何与此表中前面值都不匹配且返回值为String或void的返回值将被视为视图名称（通过RequestToViewNameTranslator选择默认视图名称） |



### 类型转换

如果参数声明为String以外的其他参数，则表示基于String的请求输入的某些带注解的控制器方法参数（例如@ RequestParam，@ RequestHeader，@ PathVariable，@ MatrixVariable和@CookieValue）可能需要进行类型转换。



对于此类情况，将根据配置的转换器自动应用类型转换。默认情况下，支持简单类型（int，long，Date等）。可以通过WebDataBinder自定义类型转换器或使用FormattingConversionService注册Formatters）。



### 矩阵变量（Matrix Variables）

矩阵变量可以出现在任何路径段中，每个变量用分号分隔，多个值用逗号分隔。如：`/cars;color=red,green;year=2012`。

也可以通过重复的变量名来指定多个值。

如：`color=red;color=green;color=blue`。



如果URL预计包含矩阵变量，控制器方法的请求映射必须使用URI变量来屏蔽该变量内容，并确保请求可以独立于矩阵变量顺序和存在而成功匹配。下面是一个使用矩阵变量的例子：

```java
// GET /pets/42;q=11;r=22

@GetMapping("/pets/{petId}")
public void findPet(@PathVariable String petId, @MatrixVariable int q) {

    // petId == 42
    // q == 11
}
```

鉴于所有路径段可能包含矩阵变量，有时可能需要消除矩阵变量预期所在的路径变量的歧义。下面演示该如何做：

```java
// GET /owners/42;q=11/pets/21;q=22

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable(name="q", pathVar="ownerId") int q1,
        @MatrixVariable(name="q", pathVar="petId") int q2) {

    // q1 == 11
    // q2 == 22
}
```

矩阵变量可以定义为可选，并指定默认值，下面演示如何做：

```java
// GET /pets/42

@GetMapping("/pets/{petId}")
public void findPet(@MatrixVariable(required=false, defaultValue="1") int q) {
    // q == 1
}
```

要获取所有矩阵变量，可以使用MultiValueMap，来看下面的例子：

```java
// GET /owners/42;q=11;r=12/pets/21;q=22;s=23

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable MultiValueMap<String, String> matrixVars,
        @MatrixVariable(pathVar="petId") MultiValueMap<String, String> petMatrixVars) {

    // matrixVars: ["q" : [11,22], "r" : 12, "s" : 23]
    // petMatrixVars: ["q" : 22, "s" : 23]
}
```

> 请注意，需要启用矩阵变量的使用。@MatrixVariable注解的使用是有歧义的，默认是关闭的。
>
> 
>
> 在MVC Java配置中，通过路径匹配将UrlPathHelper的removeSemicolonContent 设置为false。
>
> ```java
> @Configuration
> public class CustomMvcConfiguration implements InitializingBean {
> 	@Autowired
> 	private RequestMappingHandlerMapping requestMappingHandlerMapping;
> 
> 	@Override
> 	public void afterPropertiesSet() throws Exception {
> 	requestMappingHandlerMapping.setRemoveSemicolonContent(false);
> 	}
> }
> ```
>
> 
>
> 在MVC XML命名空间中，可以设置`<mvc：annotation-driven enable-matrix-variables ="true"/>`。



### @RequestParam

使用@RequestParam注解将Servlet请求参数（即查询参数或表单数据）绑定到控制器中的方法参数。

示例：

```java
@Controller
@RequestMapping("/pets")
public class EditPetForm {
    @GetMapping
    public String setupForm(@RequestParam("petId") int petId, Model model) { 
        Pet pet = this.clinic.loadPet(petId);
        model.addAttribute("pet", pet);
        return "petForm";
    }

}
```

默认情况下，对控制器方法参数使用这个注解后，该方法参数是必须的传入的。但是可以通过指定该注解的 `required` 属性为 `false` 来表明这个参数是可选的。

如果目标控制器方法参数类型不是String，则会自动进行类型转换。

将参数类型声明为数组或列表允许为同一参数名称解析多个参数值。

当@RequestParam注解方法参数声明为 `Map<String,String>`（一个key对应一个值)）或 `MultiValueMap<String,String>`（一个key对应多个值） 时，如果注解中未指定参数名称，则会使用每个给定参数名称的请求参数值填充映射。请求参数处，不能同时接收两个Map类型的参数。



### @RequestHeader

可以使用 `@RequestHeader` 注解绑定一个请求头到控制器中方法参数。

考虑如下请求头：

```
Host                    localhost:8080
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language         fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding         gzip,deflate
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300
```

下面的例子演示如何获取 `Accept-Encoding` 和 `Keep-Alive` 头：

```java
@GetMapping("/headers")
public String handle(@RequestHeader("Accept-Encoding") String encoding,
                     @RequestHeader("Keep-Alive") long keepAlive,
                     @RequestHeader("Host") String host,
                     @RequestHeader("Accept") String accept) {
    return "requestHeader->encoding:" + encoding + ", keepAlive:" + keepAlive + ", Host:" + host + ",Accept:" + accept;
}
```

如果目标方法参数不是一个 `String` 类型，也会进行自动的类型转换。

当在`Map<String,String>`，`MultiValueMap<String,String>`或`HttpHeaders`参数上使用`@RequestHeader`注解时，会将所有请求头值填充参数。

> 内置支持可用于将逗号分隔的字符串转换为字符串或类型转换系统已知的其他类型的数组或集合。
>
> 例如，使用`@RequestHeader("Accept")`注解的方法参数可以是String类型，也可以是`String[]`或`List<String>`。?



### @CookieValue

可以使用`@CookieValue`注解将HTTP cookie的值绑定到控制器中的方法参数。

考虑如下Cookie：

```
JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84
```

下面例子展示如何获取cookie的值：

```java
@GetMapping("/jsessionid")
public String handler(@CookieValue("JSESSIONID") String cookie) {
    return "cookieValue->jsessionid:" + cookie;
}
```

如果目标方法参数不是一个 `String` 类型，也会进行自动的类型转换。



### @ModelAttribute

在控制器的处理器方法参数上添加 `@ModelAttribute` 注解可以访问模型中的属性，如果不存在这个模型，则会自动将其实例化，产生一个新的模型。模型属性还覆盖了来自 HTTP Servlet 请求参数的名称与字段名称匹配的值，也就是请求参数如果和模型类中的域变量一致，则会自动将这些请求参数绑定到这个模型对象，这被称为数据绑定，从而避免了解析和转换每个请求参数和表单字段这样的代码。 



处理器方法中@ModelAttribute标注的参数会被从以下几个来源进行匹配绑定：

- 已经定义过的模型方法（带有 `@ModelAttribute` 的方法）
- HTTP Session 中和字段名匹配的会话方法（带有 `@SessionAttribute` 的方法，和模型方法类似，只是作用域不同）
- 经过 URL 转换器解析过的路径变量
- 该模型类的默认构造方法
- 调用具有与 Servlet 请求参数匹配的参数的 “主构造函数”; 参数名称通过 Java Beans `@ConstructorProperties` 或通过字节码中的运行时保留参数名称确定。



虽然一般都是使用模型方法 Model method 来使用属性填充模型，但另一种方法是依靠 `Converter<String,T>` 识别 URI 路径变量来绑定。



在下面的例子中，模型属性名称 “user” 与 URI 路径变量 “user” 匹配，并且通过将 String 类型的用户名交给给已注册的 `Converter<String,User>` 这个转换器来生成创建模型：

```java
@PutMapping("/users/{user}")
public String saveUser(@ModelAttribute("user") User user) {
    // ...
}
```

在获得模型属性实例之后，请求数据就会被绑定到模型属性上。 `WebDataBinder` 负责将 Servlet 请求参数名称（查询参数或表单字段）和目标模型对象上的字段名称进行匹配。 必要时会将属性的类型进行转换后再填充对应字段。



数据绑定不能保证不会出错，发生错误时默认情况下会抛出 `BindException` 异常，但要在处理器方法中识别出这些错误，需要在 @ModelAttribute 后面添加一个 `BindingResult` 类型的参数，需要注意的是：这个参数必须和模型属性参数 (`@ModelAttribute` 参数)相邻，如下所示：

```java
@PostMapping("/owners/{componyId}/departments/{departmentId}/edit")
public String processSubmit(@ModelAttribute("compony") Compony compony, BindingResult result) {
    if (result.hasErrors()) {
        return "componyForm";
    }
}
```

这个例子表示如果用户提交的表单不符合预期的匹配规则，就会返回视图 `componyForm`。



有时候我们需要获得一个不带数据绑定的模型属性，也就是需要在处理器方法中使用 `new` 关键字来实例化一个对象。但是在 Spring MVC 中就不用这么麻烦了，我们可以将模型注入控制器并直接访问它，或者可以添加 `@ModelAttribute（binding = false）` 来表示不需要绑定数据，如下所示：

```java
@ModelAttribute
public UserForm setUpForm() {
    return new UserForm();
}

@ModelAttribute
public User findUser(@PathVariable String userId) {
    return userRepository.findOne(userId);
}

@PostMapping("update")
public String update(@Valid UserUpdateForm form, BindingResult result,
        @ModelAttribute(binding=false) User user) {
    // ...
}
```

在参数上添加 `javax.validation.Valid` 注解或 Spring 的 `@Validated` 注解，就可以在数据绑定后使用字段校验功能了，就像这样：

```java
@PostMapping("/componies/{componyId}/departments/{departmentId}/edit")
public String processSubmit(@Valid @ModelAttribute("department") Department department, BindingResult result) {
    if (result.hasErrors()) {
        return "departmentForm";
    }
    // ...
}
```

这样写和在方法体中写 `model.addAttribute("compony",compony)` 是等价的。

需要注意的是 `@ModelAttribute` 注解如果不加，按照 `BeanUtils` 中的 `isSimpleProperty`方法来判断，如果不属于简单类型的参数，都会被自动视为 `ModelAttribute`。



> 数据绑定指的是Servlet请求绑定到模型数据，而不包含模型方法。@ModelAttribute的binding属性只对数据绑定生效。



### @SessionAttributes

用于在多个请求之间的HTTP Servlet会话中存储模型数据。是一个类级别的注解，用于声明特定控制器使用的会话属性。这通常列出模型属性的名称或模型属性的类型，这些属性应该透明地存储在会话中以供后续访问请求使用。

```java
@RestController
@RequestMapping("/sessionAttributes")
@SessionAttributes("name")
public class HandlerMethodSessionAttributesController {
    // ...
}
```

在第一个请求中，当名称为pet的模型属性添加到模型中时，它会自动保存在HTTP Servlet会话中。它保持不变，直到另一个控制器方法添加相同的模型属性到模型中的时候会替换它，或者使用SessionStatus方法参数来清除存储。只对当前控制类起作用。

```java
@RestController
@RequestMapping("/sessionAttributes")
@SessionAttributes("name")
public class HandlerMethodSessionAttributesController {

    @RequestMapping("/add")
    public String addToSessionAttribute(Model model) {
        model.addAttribute("name", "tom");
        return "sessionAttributes->add->name:tom";
    }
    
    @RequestMapping("/clear")
    public String removeSessionAttributes(HttpSession session, SessionStatus status) {
        String name = session.getAttribute("name").toString();
        status.setComplete();
        return "sessionAttributes->clear,name=" + name;
    }
}
```



### @SessionAttribute

如果需要访问全局管理的预先存在的会话属性（例如，在控制器外部 - 例如，通过过滤器），可能存在也可能不存在，可以在方法参数上使用@SessionAttribute注解。

```java
@RequestMapping("/")
public String handle(@SessionAttribute User user) { 
    // ...
}
```

对于需要添加或删除会话属性，考虑将`org.springframework.web.context.request.WebRequest`或`javax.servlet.http.HttpSession`注入控制器方法。

要在会话中临时存储模型属性作为控制器工作流的一部分，请考虑使用@SessionAttributes。



### @RequestAttribute

与`@SessionAttribute`类似，可以使用`@RequestAttribute`注解来访问先前创建的预先存在的请求属性（例如，通过Servlet Filter或 HandlerInterceptor）：

```java
@GetMapping("/")
public String handle(@RequestAttribute Client client) { 
    // ...
}
```



### Redirect Attributes

默认情况下，所有模型属性都被视为在重定向URL中作为URI模板变量公开。在其余属性中，原始类型或集合或基本类型数组的属性会自动附加为查询参数。



如果专门为重定向准备了模型实例，则将原始类型属性作为查询参数附加可以是期望的结果。但是，在带注解的控制器中，模型可以包含为渲染目的而添加的其他属性（例如，下拉字段值）。为了避免在URL中出现此类属性的可能性，`@RequestMapping`方法可以声明RedirectAttributes类型的参数，并使用它来指定可供RedirectView使用的确切属性。如果方法重定向，则使用RedirectAttributes的内容。否则，使用模型的内容。



RequestMappingHandlerAdapter提供了一个名为ignoreDefaultModelOnRedirect的标志，您可以使用该标志指示如果控制器方法重定向，则永远不应使用默认模型的内容。相反，控制器方法应声明RedirectAttributes类型的属性，如果不这样做，则不应将任何属性传递给RedirectView。MVC命名空间和MVC Java配置都将此标志设置为false，以保持向后兼容性。但是，对于新应用程序，我们建议将其设置为true。



请注意，在展开重定向URL时，将自动使当前请求中的URI模板变量可用，并且您不需要通过Model或RedirectAttributes显式添加它们。

以下示例显示如何定义重定向：

```java
@PostMapping("/files/{path}")
public String upload(...) {
    // ...
    return "redirect:files/{path}";
}
```

将数据传递到重定向目标的另一种方法是使用flash属性。与其他重定向属性不同，Flash属性保存在HTTP会话中（因此，不会出现在URL中）。



### Flash Attributes

Flash属性为一个请求提供了一种存储打算在另一个请求中使用的属性的方法。重定向时最常需要这种方法 - 例如，Post-Redirect-Get模式。Flash重定向（通常在会话中）之前临时保存Flash属性，以便在重定向后对请求可用并立即删除。

Spring MVC有两个主要的抽象支持flash属性。FlashMap用于保存Flash属性，而FlashMapManager用于存储，检索和管理FlashMap实例。



Flash属性支持始终处于“打开”状态，无需显式启用。但是，如果不使用，它永远不会导致HTTP会话创建。在每个请求中，都有一个“输入”FlashMap，其中包含从先前请求（如果有）传递的属性，以及一个“输出”FlashMap，其中包含要为后续请求保存的属性。两个FlashMap实例都可以通过RequestContextUtils中的静态方法从Spring MVC中的任何位置访问。



带注解的控制器通常不需要直接使用FlashMap。相反，@RequestMapping方法可以接受RedirectAttributes类型的参数，并使用它为重定向场景添加flash属性。通过RedirectAttributes添加的Flash属性会自动传播到“输出”FlashMap。同样，在重定向之后，“输入”FlashMap的属性会自动添加到为目标URL提供服务的控制器的模型中。



> **匹配请求到flash属性**
>
> Flash属性的概念存在于许多其他Web框架中，并且已经证明有时会暴露于并发问题。这是因为，根据定义，Flash属性将被存储直到下一个请求。但是，非常“下一个”请求可能不是预期的接收者而是另一个异步请求（例如，轮询或资源请求），在这种情况下，过早删除flash属性。
>
> 
>
> 为了减少此类问题的可能性，RedirectView使用目标重定向URL的路径和查询参数自动“标记”FlashMap实例。反过来，默认的FlashMapManager在查找“输入”FlashMap时将该信息与传入请求进行匹配。
>
> 
>
> 这并不能完全消除并发问题的可能性，但会使用重定向URL中已有的信息大大减少并发问题。因此，我们建议您主要使用Flash属性进行重定向方案。



### Multipart

启用MultipartResolver后，将解析具有multipart/form-data的POST请求的内容，并将其作为常规请求参数进行访问。以下示例访问一个常规表单字段和一个上传文件：

```java
@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(@RequestParam("name") String name,
            @RequestParam("file") MultipartFile file) {
        if (!file.isEmpty()) {
            byte[] bytes = file.getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
        }
        return "redirect:uploadFailure";
    }
}
```

将参数类型声明为`List<MultipartFile>`允许为同一参数名称解析多个文件。

当@RequestParam注释声明为Map`<String,MultipartFile>`或MultiValueMap`<String,MultipartFile>`时，如果没有在注解中指定参数名称，则会使用每个给定参数名称的多部分文件填充Map。

> 通过Servlet3.0多部分解析，可以使用`javax.servlet.http.Part`替代 Spring的`MultipartFile`，作为方法参数或集合值类型。



还可以将多部分内容用作绑定到命令对象的数据的一部分。例如，前面示例中的表单字段和文件可以是表单对象上的字段，如以下示例所示：

```java
class MyForm {

    private String name;

    private MultipartFile file;

    // ...
}

@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(MyForm form, BindingResult errors) {
        if (!form.getFile().isEmpty()) {
            byte[] bytes = form.getFile().getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
        }
        return "redirect:uploadFailure";
    }
}
```

还可以在RESTful服务方案中从非浏览器客户端提交多部分请求。以下示例显示了带有JSON的文件：

```
POST /someUrl
Content-Type: multipart/mixed

--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
Content-Disposition: form-data; name="meta-data"
Content-Type: application/json; charset=UTF-8
Content-Transfer-Encoding: 8bit

{
    "name": "value"
}
--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
Content-Disposition: form-data; name="file-data"; filename="file.properties"
Content-Type: text/xml
Content-Transfer-Encoding: 8bit
... File Data ...
```

您可以使用@RequestParam作为String访问“元数据”部分，但您可能希望它从JSON反序列化（类似于@RequestBody）。在使用HttpMessageConverter转换后，使用@RequestPart注解访问多部分：

```java
@PostMapping("/")
public String handle(@RequestPart("meta-data") MetaData metadata,
        @RequestPart("file-data") MultipartFile file) {
    // ...
}
```

可以将@RequestPart与javax.validation.Valid结合使用或使用Spring的@Validated注释，这两种注释都会导致应用标准Bean验证。默认情况下，验证错误会导致MethodArgumentNotValidException，并将其转换为400（BAD_REQUEST）响应。或者，您可以通过Errors或BindingResult参数在控制器内本地处理验证错误，如以下示例所示：

```java
@PostMapping("/")
public String handle(@Valid @RequestPart("meta-data") MetaData metadata,
        BindingResult result) {
    // ...
}
```



### @RequestBody

您可以使用@RequestBody注解通过HttpMessageConverter将请求主体读取并反序列化为Object。以下示例使用@RequestBody参数：

```java
@PostMapping("/accounts")
public void handle(@RequestBody Account account) {
    // ...
}
```

您可以使用MVC配置的“Message Converters ”选项来配置或自定义消息转换。

您可以将@RequestBody与javax.validation.Valid或Spring的@Validated注释结合使用，这两种注释都会导致应用标准Bean验证。默认情况下，验证错误会导致MethodArgumentNotValidException，并将其转换为400（BAD_REQUEST）响应。或者，您可以通过Errors或BindingResult参数在控制器内本地处理验证错误，如以下示例所示：

```java
@PostMapping("/accounts")
public void handle(@Valid @RequestBody Account account, BindingResult result) {
    // ...
}
```



### HttpEntity

HttpEntity与使用@RequestBody或多或少相同，但它基于一个公开请求标题和正文的容器对象。

```java
@PostMapping("/accounts")
public void handle(HttpEntity<Account> entity) {
    // ...
}
```



### @ResponseBody

您可以在方法上使用@ResponseBody批注，以通过HttpMessageConverter将返回序列化到响应主体。

```java
@GetMapping("/accounts/{id}")
@ResponseBody
public Account handle() {
    // ...
}
```

类级别也支持@ResponseBody，在这种情况下，它由所有控制器方法继承。这是@RestController的效果，它只不过是一个用@Controller和@ResponseBody标记的元注释。

您可以使用MVC配置的“Message Converters ”选项来配置或自定义消息转换。

可以将@ResponseBody方法与JSON序列化视图结合使用。参见：[Jackson JSON](https://docs.spring.io/spring/docs/5.1.8.RELEASE/spring-framework-reference/web.html#mvc-ann-jackson)



### ResponseEntity

ResponseEntity与@ResponseBody类似，但具有响应状态码和响应标头。

```java
@GetMapping("/something")
public ResponseEntity<String> handle() {
    String body = ... ;
    String etag = ... ;
    return ResponseEntity.ok().eTag(etag).build(body);
}
```



### Jackson JSON

Spring为Jackson JSON库提供支持。

JSON 视图

Spring MVC为Jackson的序列化视图提供内置支持，允许仅渲染Object中所有字段的子集。要将其与@ResponseBody或ResponseEntity控制器方法一起使用，您可以使用Jackson的@JsonView批注来激活序列化视图类，如以下示例所示：

```java
@RestController
public class UserController {

    @GetMapping("/user")
    @JsonView(User.WithoutPasswordView.class)
    public User getUser() {
        return new User("eric", "7!jd#h23");
    }
}

public class User {

    public interface WithoutPasswordView {};
    public interface WithPasswordView extends WithoutPasswordView {};

    private String username;
    private String password;

    public User() {
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @JsonView(WithoutPasswordView.class)
    public String getUsername() {
        return this.username;
    }

    @JsonView(WithPasswordView.class)
    public String getPassword() {
        return this.password;
    }
}
```

> @JsonView允许一组视图类，但每个控制器方法只能指定一个。如果需要激活多个视图，可以使用复合接口。

对于依赖于视图分辨率的控制器，可以将序列化视图类添加到模型中，如以下示例所示：

```java
@Controller
public class UserController extends AbstractController {

    @GetMapping("/user")
    public String getUser(Model model) {
        model.addAttribute("user", new User("eric", "7!jd#h23"));
        model.addAttribute(JsonView.class.getName(), User.WithoutPasswordView.class);
        return "userView";
    }
}
```



## 模型（Model）





## 数据绑定（DataBinder）



## 异常处理（Exceptions）

### 方法参数



### 返回值



### REST API exceptions





## Controller Advice



