# 带注解的Controller

Spring MVC提供了一个基于注解的编程模型，其中`@Controller`和`@RestController`组件使用注解来表达请求映射，请求输入，异常处理等等。带注解的Controller具有灵活的方法签名，不必扩展基类，也不必实现特定的接口。例子如下：

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

在Servlet的`WebApplicationContext`中，你可以使用Spring bean definition定义一个Controller Bean。@Controller构造型允许自动检测，与Spring一致支持，以检测类路径中的@Component类并自动注册它们的bean定义。它还充当带注解的类的构造型，表明它作为Web组件的角色。

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

@RestController是一个由@Controller和@ResponseBody元注解组成的组合注解，指示控制器内每个方法继承类型级别@ResponseBody注解，因此，它直接写入响应主体，而不是视图解析，并使用HTML模板呈现。

### AOP代理

在某些情况下，您可能需要在运行时用AOP代理来修饰控制器。一个例子是，如果您选择直接在控制器上使用@Transactional注释。在这种情况下，特别是对于控制器，我们建议使用基于类的代理。这通常是控制器的默认选择。但是，如果控制器必须实现一个不是Spring上下文回调的接口（如InitializingBean、*Aware和其他），则可能需要显式配置基于类的代理。例如，使用`<tx:annotationdriven/>`，您可以更改为`<tx:annotation-driven proxy-target-class="true"/>`。

## 请求映射

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

可以根据请求的`Content-Type`缩小请求映射，如下：

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

可以根据`Accept`请求头和控制器方法生成的内容类型列表来缩小请求映射，如下：

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

可以根据请求参数条件缩小请求映射范围。测试是否存在请求参数（myParam），不存在（!myParam）或者特定值（myParam=myValue）。

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

