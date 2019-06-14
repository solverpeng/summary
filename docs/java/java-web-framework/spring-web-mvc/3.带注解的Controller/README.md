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



### @CookieValue



### @ModelAttribute



### @SessionAttributes



### @SessionAttribute



### @RequestAttribute



### Redirect Attributes



### Flash Attributes



### Multipart



### @RequestBody



### HttpEntity



### @ResponseBody



### ResponseEntity



### Jackson JSON



## 模型（Model）





## 数据绑定（DataBinder）



## 异常处理（Exceptions）

### 方法参数



### 返回值



### REST API exceptions





## Controller Advice



