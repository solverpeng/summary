# 自定义JSTL标签和函数库

## 自定义JSTL标签

### 编写标签处理类

#### 实现 SimpleTag 接口

通过 `setJspContext()`方法可以获取到 `jspContext` 对象，实际上也是 `pageContext` 对象。

在 `doTag()` 方法中完成逻辑，通过 `JspWriter out = jspContext.getOut();` 获取到的 `out` 对象，可以输出到页面。如：

```java
public class MyTag2 implements SimpleTag {
    private JspContext jspContext = null;
    @Override
    public void doTag() throws JspException, IOException {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
        String dataStr = simpleDateFormat.format(new Date());
        JspWriter out = jspContext.getOut();
        out.write(dataStr);
    }

    @Override
    public void setParent(JspTag jspTag) {

    }

    @Override
    public JspTag getParent() {
        return null;
    }

    @Override
    public void setJspContext(JspContext jspContext) {
        this.jspContext = jspContext;
    }

    @Override
    public void setJspBody(JspFragment jspFragment) {

    }
}
```

#### 在 WEB-INF 下新建自定义 tld 文件

可以参考 `JSTL` 本身一些标签的的实现，如 `c.tld`。然后注册标签。如：

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-jsptaglibrary_2_0.xsd"
        version="2.0">

    <display-name>solverpeng core</display-name>
    <tlib-version>1.1</tlib-version>
    <short-name>solverpeng</short-name>
    <uri>http://solverpeng.com/tags</uri>

    <tag>
        <name>showDate</name>
        <tag-class>com.nucsoft.myservlet.tag.MyTag2</tag-class>
        <body-content>empty</body-content>
    </tag>
    
</taglib>
```

#### 在页面中导入自定义的标签库，然后使用

```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="s" uri="http://solverpeng.com/tags" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <s:showDate/>
</body>
</html>
```

### 继承 SimpleTagSupport

其中在 `SimpleTagSupport` 类中已经完成了对 `JspTag、JspContext，JspFragment` 的注入操作，可以直接使用。

### 需要注意的地方

在自定义标签`tld`文件中，`tag` 节点里的 `body-content` 节点内容有四个可选值，分别介绍一下：

1. tagdependent：将标签体的内容不经过任何修改和处理原封不动的交个标签处理类
2. empty：没有任何标签体
3. scriptless：表示标签体支持纯文本、EL表达式、JSTL等标签，不支持JSP脚本片段和JSP表达式
4. JSP：在 SimpleTag 标签体系下不可用

可以通过 `jspContext` 获取到其他8大 `JSP` 隐含对象。

如果为带属性的自定义标签，在目标处理类中通过提供 `setXxx()` 方法来获取，在 `tld` 文件中，需要在 `tag` 元素中添加子节点 `attribute` 。

其中对 `attribute` 节点下的三个子节点进行说明：

1. name：属性名称，和目标处理类中通过 setXxx() 方法定义的属性名一致
2. required：是否是必须输入的
3. rtexprvalue：是否支持 EL 表达式

如果需要对标签体处理，需要在目标处理类中进行一系列的处理：

1. 在 doTag() 方法中通过 this.getJspBody() 可以获取到 JspFragment 对象
2. 执行 JspFragment 对象的 invoke() 方法，若不需要对执行后的标签体进行处理，而是直接输出的话，invoke() 方法的参数传入 null就 ok。

若是需要对执行后的标签体进行处理，则需要传入一个 `Write` 对象，在调用 `invoke()` 方法之后，然后对 `Write` 对象进行处理。如：

```java
JspFragment jspBody = this.getJspBody();
StringWriter out = new StringWriter();
jspBody.invoke(out);
String content = out.toString();
String result = content.toUpperCase();
JspWriter writer = this.getJspContext().getOut();
writer.print(result);
```

## 自定义JSTL函数库

### 定义函数处理类

要求类和方法必须为 `public` ，且方法必须为 `static` 。如：

```java
public class MyFunction {
    public static String hello() {
        return new Date().toString();
    }
}
```

### 在 WEB-INF 下新建自定义 tld 文件

可以参考 `JSTL` 本身一些标签的的实现，如 `c.tld`。然后注册函数。如：

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-jsptaglibrary_2_0.xsd"
        version="2.0">
    <display-name>solverpeng core</display-name>
    <tlib-version>1.1</tlib-version>
    <short-name>solverpeng</short-name>
    <uri>http://solverpeng.com/tags</uri>

    <function>
        <name>hello</name>
        <function-class>com.nucsoft.myservlet.function.MyFunction</function-class>
        <function-signature>java.lang.String hello()</function-signature>
    </function>
</taglib>
```

### 在JSP页面中导入自定义的函数库，然后在页面中使用

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="s" uri="http://solverpeng.com/tags" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    ${s:hello()}
</body>
</html>
```

## JSTL中一个不常用的标签 c:forTokens
```html
<% pageContext.setAttribute("str", "aa,bb,ee,mm,tt"); %>
<c:forTokens items="${str }" delims="," var="item">
${item }
</c:forTokens>
```