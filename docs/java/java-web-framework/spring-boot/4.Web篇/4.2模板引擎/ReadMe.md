# Thymeleaf模版引擎

Thymeleaf是一个XML/XHTML/HTML5模板引擎，可用于Web与非Web环境中的应用开发。



Spring Boot提供了 Thymeleaf 的启动器，使用时在 pom 文件中引入启动器依赖即可。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

Spring Boot-2.1.5.RELEASE 版本仲裁中心默认 Thymeleaf 版本信息如下：

```xml
<thymeleaf.version>3.0.11.RELEASE</thymeleaf.version>
<thymeleaf-extras-data-attribute.version>2.0.1</thymeleaf-extras-data-attribute.version>
<thymeleaf-extras-java8time.version>3.0.4.RELEASE</thymeleaf-extras-java8time.version>
<thymeleaf-extras-springsecurity.version>3.0.4.RELEASE</thymeleaf-extras-springsecurity.version>
<thymeleaf-layout-dialect.version>2.3.0</thymeleaf-layout-dialect.version>
```

注意：

`thymeleaf.version` 3版本需要 `thymeleaf-layout-dialect.version` 最低为2版本。

`thymeleaf.version` 2版本需要 `thymeleaf-layout-dialect.version` 为1版本。



## Spring Boot下使用 Thymeleaf

引入了 `spring-boot-starter-thymeleaf` 后，默认就可以使用 `ThymeleafAutoConfiguration` 和 `ThymeleafProperties` 进行配置。

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

	private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;

	public static final String DEFAULT_PREFIX = "classpath:/templates/";

	public static final String DEFAULT_SUFFIX = ".html";
}
```

可以通过在主配置类中 `spring.thymeleaf` 前缀对 Thymeleaf 进行配置。

同样可以看到，使用 Thymeleaf 后，默认会使用 `ThymeleafViewResolver` 视图解析器。默认前缀为：`classpath:/templates/`，默认后缀为：`.html`。



默认情况下，Thymeleaf 的缓存是开启的，在开发阶段关闭缓存。在著配置类中添加如下配置信息：`spring.thymeleaf.cache=false`。



### 示例

在项目类路径的 `templates` 目录下新增 `success.html` 文件，新增 `success` 请求，返回 `success` 视图名。

项目结构：

![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/github/java-summary/springboot/thymeleaf-1.png)

success.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    成功页面
</body>
</html>
```

Handler

```java
@Controller
public class HelloController {
    @RequestMapping("/success")
    public String success() {
        return "success";
    }
}
```

请求：http://localhost:8080/success

成功可以访问到 success 页面。



## Thymeleaf 语法

### 引入约束

首先要在 Thymeleaf 模版中引入 Thymeleaf 约束。如下：

`<html lang="en" xmlns:th="http://www.thymeleaf.org">`。

引入约束后，在使用 Thymeleaf 标签的时候，就会有友好提示。

### 常用标签

| 关键字      | 功能介绍                                     | 案例                                                         |
| ----------- | -------------------------------------------- | ------------------------------------------------------------ |
| th:id       | 替换id                                       | <input th:id="'xxx' + ${collect.id}"/>                       |
| th:text     | 文本替换                                     | <p th:text="${collect.description}">description</p>          |
| th:utext    | 支持html的文本替换                           | <p th:utext="${htmlcontent}">content</p>                     |
| th:object   | 替换对象                                     | <div th:object="${session.user}">                            |
| th:value    | 属性赋值                                     | <input th:value="${user.name}" />                            |
| th:with     | 变量赋值运算                                 | <div th:with="isEven=${prodStat.count}%2==0"></div>          |
| th:style    | 设置样式                                     | th:style="'display:' + @{(${sitrue} ? 'none' : 'inline-block')} + ''" |
| th:onclick  | 点击事件                                     | th:onclick="'getCollect()'"                                  |
| th:each     | 遍历                                         | <tr th:each="user,userStat:${users}">                        |
| th:if       | 判断条件                                     | <a th:if="${userId == collect.userId}" >                     |
| th:unless   | 和th:if判断相反                              | <a th:href="@{/login}" th:unless=${session.user != null}>Login</a> |
| th:href     | 链接地址                                     | <a th:href="@{/login}" th:unless=${session.user != null}>Login</a> |
| th:switch   | 多路选择 配合th:case 使用                    | <div th:switch="${user.role}">                               |
| th:case     | th:switch的一个分支                          | <p th:case="'admin'">User is an administrator</p>            |
| th:fragment | 布局标签，定义一个代码片段，方便其它地方引用 | <div th:fragment="alert">                                    |
| th:include  | 布局标签，替换内容到引入的文件               | <head th:include="layout :: htmlhead" th:with="title='xx'"></head> |
| th:replace  | 布局标签，替换整个标签到引入的文件           | <div th:replace="fragments/header :: title"></div>           |
| th:selected | selected选择框 选中                          | th:selected="(${xxx.id} == ${configObj.dd})"                 |
| th:src      | 图片类地址引入                               | <img class="img-responsive" alt="App Logo" th:src="@{/img/logo.png}" /> |
| th:inline   | 定义js脚本可以使用变量                       | <script type="text/javascript" th:inline="javascript">       |
| th:action   | 表单提交的地址                               | <form action="subscribe.html" th:action="@{/subscribe}">     |
| th:remove   | 删除某个属性                                 | <tr th:remove="all"> <br />1.all:删除包含标签和所有的孩子。<br />2.body:不包含标记删除,但删除其所有的孩子。<br />3.tag:包含标记的删除,但不删除它的孩子。<br />4.all-but-first:删除所有包含标签的孩子,除了第一个。<br />5.none:什么也不做。这个值是有用的动态评估。 |
| th:attr     | 设置标签属性，多个属性可以用逗号分隔         | 比如 th:attr="src=@{/image/aa.jpg},title=#{logo}"，此标签不太优雅，一般用的比较少。 |

### 标准表达式语法

* `$`：变量表达式
* `*`：选择表达式
* `#`：国际化表达式
* `@`：URL表达式

#### 变量表达式

变量表达式即OGNL表达式或Spring EL表达式。如：`${session.user.name}`

#### 选择表达式

选择表达式很像变量表达式，不过它可以用 `*` 来代替上下文变量来执行。如：

被指定的object由`th:object`属性定义：

```xml
<div th:object="${book}">  
    ...  
    <span th:text="*{title}">...</span>  
    ...  
</div>
```

#### 国际化表达式

国际化表达式允许我们从一个外部文件获取区域文字信息(.properties)，用Key索引Value，还可以提供一组参数(可选)。

```xml
<table>  
    ...  
    <th th:text="#{header.address.city}">...</th>  
    <th th:text="#{header.address.country}">...</th>  
    ...  
</table>  
```

#### URL表达式

URL表达式指的是把一个有用的上下文或会话信息添加到URL，这个过程经常被叫做URL重写。

如：`@{/order/list}`，URL还可以设置参数： `@{/order/details(id=${orderId})}`，相对路径：`@{../documents/report}`。

又如：

```xml
<form th:action="@{/createOrder}">
    <a href="main.html" th:href="@{/main}">
```

### 表达式支持的语法

#### 字面量（Literals）

- 文本文字（Text literals）: `'one text', 'Another one!',…`
- 数字文本（Number literals）: `0, 34, 3.0, 12.3,…`
- 布尔文本（Boolean literals）: `true, false`
- 空（Null literal）: `null`
- 文字标记（Literal tokens）: `one, sometext, main,…`

#### 文本操作（Text operations）

- 字符串连接(String concatenation): `+`
- 文本替换（Literal substitutions）: `|The name is ${name}|`

#### 算术运算（Arithmetic operations）

- 二元运算符（Binary operators）: `+, -, *, /, %`
- 减号（单目运算符）Minus sign (unary operator): `-`

#### 布尔操作（Boolean operations）

- 二元运算符（Binary operators）:`and, or`
- 布尔否定（一元运算符）Boolean negation (unary operator):`!, not`

#### 比较和等价(Comparisons and equality)

- 比较（Comparators）: `>, <, >=, <= (gt, lt, ge, le)`
- 等值运算符（Equality operators）:`==, != (eq, ne)`

#### 条件运算符（Conditional operators）

- If-then: `(if) ? (then)`
- If-then-else: `(if) ? (then) : (else)`
- Default: (value) ?: `(defaultvalue)`



所有这些特征都可以被组合嵌套。如下：

```
'User is of type ' + (${user.isAdmin()} ? 'Administrator' : (${user.type} ?: 'Unknown'))
```



### 常见用法



#### 赋值与字符串拼接

```xml
<p  th:text="${collect.description}">description</p><span th:text="'Welcome to our application, ' + ${user.name} + '!'">
```

字符串拼接还有另外一种简洁的写法：

```xml
<span th:text="|Welcome to our application, ${user.name}!|">
```

#### 条件判断 If/Unless

Thymeleaf中使用th:if和th:unless属性进行条件判断，下面的例子中，`<a>`标签只有在`th:if`中条件成立时才显示：

```xml
<a th:if="${myself=='yes'}" ></a><a th:unless=${session.user != null} th:href="@{/login}" >Login</a>
```

th:unless于th:if恰好相反，只有表达式中的条件不成立，才会显示其内容。



#### for 循环

 ```xml
<tr  th:each="collect,iterStat : ${collects}">
	<th scope="row" th:text="${collect.id}">1</th>
	<td ><img th:src="${collect.webLogo}"/></td>
	<td th:text="${collect.url}">Mark</td>
	<td th:text="${collect.title}">Otto</td>
	<td th:text="${collect.description}">@mdo</td>
	<td th:text="${terStat.index}">index</td>
</tr>
 ```

**iterStat** 称作状态变量，属性有：

- index:当前迭代对象的index（从0开始计算）
- count: 当前迭代对象的index(从1开始计算)
- size:被迭代对象的大小
- current:当前迭代变量
- even/odd:布尔值，当前循环是否是偶数/奇数（从0开始计算）
- first:布尔值，当前循环是否是第一个
- last:布尔值，当前循环是否是最后一个

#### URL

Thymeleaf对于URL的处理是通过语法 `@{…}` 来处理的。如果需要Thymeleaf对URL进行渲染，那么务必使用th:href，th:src等属性，下面是一个例子：

```xml
<!-- Will produce 'http://localhost:8080/standard/unread' (plus rewriting) -->
<a  th:href="@{/standard/{type}(type=${type})}">view</a>
<!-- Will produce '/gtvg/order/3/details' (plus rewriting) -->
<a href="details.html" th:href="@{/order/{orderId}/details(orderId=${o.id})}">view</a>
```

设置背景

```xml
<div th:style="'background:url(' + @{/<path-to-image>} + ');'"></div>
```

根据属性值改变背景

```xml
 <div class="media-object resource-card-image"  th:style="'background:url(' + @{(${collect.webLogo}=='' ? 'img/favicon.png' : ${collect.webLogo})} + ')'" ></div>
```

几点说明：

- 上例中URL最后的`(orderId=${o.id})` 表示将括号内的内容作为URL参数处理，该语法避免使用字符串拼接，大大提高了可读性
- `@{...}`表达式中可以通过`{orderId}`访问Context中的orderId变量
- `@{/order}`是Context相关的相对路径，在渲染时会自动添加上当前Web应用的Context名字，假设context名字为app，那么结果应该是/app/order

#### 内联JS

内联文本：`[[…]]`内联文本的表示方式，使用时，必须先用`th:inline=”text/javascript/none”`激活，`th:inline`可以在父级标签内使用，甚至作为body的标签。内联文本尽管比th:text的代码少，不利于原型显示。

```xml
<script th:inline="javascript">
/*<![CDATA[*/
...
var username = /*[[${sesion.user.name}]]*/ 'Sebastian';
var size = /*[[${size}]]*/ 0;
...
/*]]>*/
</script>
```

#### 内嵌变量

为了模板更加易用，Thymeleaf还提供了一系列Utility对象（内置于Context中），可以通过`#`直接访问：

* dates ： `java.util.Date`的功能方法类

* calendars : 类似`#dates`，面向`java.util.Calendar`

* numbers : 格式化数字的功能方法类

* strings : 字符串对象的功能类，`contains,startWiths,prepending/appending`等等

* objects: 对`objects`的功能类操作

* bools: 对布尔值求值的功能方法

* arrays：对数组的功能类方法

* lists: 对`lists`功能类方法

* sets

* maps 

常见用法如下：

**dates：**

```xml
/*
 * Format date with the specified pattern
 * Also works with arrays, lists or sets
 */
${#dates.format(date, 'dd/MMM/yyyy HH:mm')}
${#dates.arrayFormat(datesArray, 'dd/MMM/yyyy HH:mm')}
${#dates.listFormat(datesList, 'dd/MMM/yyyy HH:mm')}
${#dates.setFormat(datesSet, 'dd/MMM/yyyy HH:mm')}

/*
 * Create a date (java.util.Date) object for the current date and time
 */
${#dates.createNow()}

/*
 * Create a date (java.util.Date) object for the current date (time set to 00:00)
 */
${#dates.createToday()}
```

**strings：**

```xml
/*
 * Check whether a String is empty (or null). Performs a trim() operation before check
 * Also works with arrays, lists or sets
 */
${#strings.isEmpty(name)}
${#strings.arrayIsEmpty(nameArr)}
${#strings.listIsEmpty(nameList)}
${#strings.setIsEmpty(nameSet)}

/*
 * Check whether a String starts or ends with a fragment
 * Also works with arrays, lists or sets
 */
${#strings.startsWith(name,'Don')}                  // also array*, list* and set*
${#strings.endsWith(name,endingFragment)}           // also array*, list* and set*

/*
 * Compute length
 * Also works with arrays, lists or sets
 */
${#strings.length(str)}

/*
 * Null-safe comparison and concatenation
 */
${#strings.equals(str)}
${#strings.equalsIgnoreCase(str)}
${#strings.concat(str)}
${#strings.concatReplaceNulls(str)}

/*
 * Random
 */
${#strings.randomAlphanumeric(count)}
```

### 参考链接

https://github.com/cloudfavorites/favorites-web