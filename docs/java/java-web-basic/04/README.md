# Servlet 编码设置

## HTTP Get 请求编码解码过程

1. HTTP GET请求流程
   - 参数直接添加到了 `url`后面，同`url`一起提交到服务器。
   - 常见格式为：`http://ip:port/path/file?parm1=v1&parm2=v2...`
   - 服务器端容器，如`Servlet`通过`ServletRequet#getParameter`获取相应的参数值
2. URL编码
   - URL规定，通过URL进行网络访问时，需要将URL参数中的特殊字符进行URL编码
   - 特殊字符
     - 出大小写字母、数字、`-_.（横线、下划线、小数点）`这三类字符之外的字符，均为特殊字符。
     - 特殊字符需要按照一定的编码格式（`utf-8`或`gbk`）转换为byte[]。
   - 将生成的byte[]，每个byte前添加`%`号，生成`URL`编码后的字符串。
3. URL解码
   - web容器，以`tomcat`为例，在接收到用户的`Get`请求后，会解析`http`请求报文，根据报文内容与服务器的环境，生成`request`对象，发送给`Servlet`处理。
   - 服务器的环境，就是当前服务器的一些配置信息，其中重要的一项就是**“默认编码格式”，以tomcat8为例，默认编码格式为utf-8。**
   - 如`Servlet`通过`ServletRequet#getParameter`获取相应的参数值，调用背后的工作：
     - 在第一次调用该方法时，`tomcat`会将`queryString`按照&和=号进行分割成一个个的键值对，
       然后对各个键和它对应的值进行`url`解码，`string str = URLDecoder.decode(键/值，“tomcat编码格式”);`，经过url编码的键/值，又回归到了原本的模样。（解码的过程与编码过程相反）。
       然后，解析该字符串，获取参数名和参数值（键值对），放入`request`对象的`parameterMap`中。
     - 第二步根据参数名，查找`parameterMap`，返回参数值。
   - 对于`GET`请求，对`URL`中参数进行`URLDecode`的工作，`tomcat`帮助我们做了，解码时采用的编码格式为`tomcat`默认的编码格式，当然我们也可以通过配置文件，修改默认编码。
4. 统一编码
   - 使用`JavaScript`对`URL`进行编码，然后向服务器提交。
     - `encodeURI()`函数，对整个`URL`进行编码。编码完成后，输出`utf-8`的形式。对应解码函数是`decodeURI()`。不对单引号`'`进行编码。
5. 统一解码
   - 修改`tomcat`的`server.xml`中的`URIEncoding`属性
   - ```str = new String(str.getBytes("iso-8859-1"), "utf-8");```
6. 说明
   - 如果decode时的编码格式与encode时不同，那么参数就会出现中文乱码的问题。
   - 不同的操作系统、不同的浏览器、不同的网页字符集，将导致完全不同的编码结果。
   - 不论编码还是解码针对的是请求参数，URI中的特殊字符并不会被解码。

## Http POST请求编码解码

`POST`用于提交表单数据，请求时，表单数据存放在消息体中。

1. 编码
   - 对于表单`enctype`为`application/x-www-form-urlencoded`（默认）的请求，编码方式如下
     - 没有用`accept-charset`指定，就用文档本身的编码
     - 设置了`accept-charset`的值，就用它设置的值
2. 解码
   - 通过`javax.servlet.ServletRequest#setCharacterEncoding`设置对客户端请求体（`post`）的解码方式，不指定的情况下，默认使用`"iso-8859-1"`。

## GET/POST区别

1. `GET`是URL解码的方式。默认解码格式是`tomcat`编码格式。
2. `POST`是实体内容解码方式。默认解码方式是`request`容器编码格式。与`tomcat`编码方式无关。

## Http 响应编码设置

1. `response.setContentType`：指定`HTTP`响应的编码,同时指定了浏览器显示的编码。
2. `response.setCharacterEncoding`：指定服务端编码格式，并告知客户端解码时的编码码格式。
3. 调用上面这两个方法，必须在`getWriter`执行之前或者`response`被提交之前。
4. `response.setCharacterEncoding`不会覆盖`response.setContentType`设置。它两一个是编码一个是解码。

## SpringMVC编码设置

`SpringMVC`提供了`org.springframework.web.filter.CharacterEncodingFilter`来统一编码。

```java
public class CharacterEncodingFilter extends OncePerRequestFilter {
    @Nullable
    private String encoding;
    private boolean forceRequestEncoding;
    private boolean forceResponseEncoding;

    public CharacterEncodingFilter() {
        this.forceRequestEncoding = false;
        this.forceResponseEncoding = false;
    }

    public CharacterEncodingFilter(String encoding) {
        this(encoding, false);
    }

    public CharacterEncodingFilter(String encoding, boolean forceEncoding) {
        this(encoding, forceEncoding, forceEncoding);
    }

    public CharacterEncodingFilter(String encoding, boolean forceRequestEncoding, boolean forceResponseEncoding) {
        this.forceRequestEncoding = false;
        this.forceResponseEncoding = false;
        Assert.hasLength(encoding, "Encoding must not be empty");
        this.encoding = encoding;
        this.forceRequestEncoding = forceRequestEncoding;
        this.forceResponseEncoding = forceResponseEncoding;
    }

    public void setEncoding(@Nullable String encoding) {
        this.encoding = encoding;
    }

    @Nullable
    public String getEncoding() {
        return this.encoding;
    }

    public void setForceEncoding(boolean forceEncoding) {
        this.forceRequestEncoding = forceEncoding;
        this.forceResponseEncoding = forceEncoding;
    }

    public void setForceRequestEncoding(boolean forceRequestEncoding) {
        this.forceRequestEncoding = forceRequestEncoding;
    }

    public boolean isForceRequestEncoding() {
        return this.forceRequestEncoding;
    }

    public void setForceResponseEncoding(boolean forceResponseEncoding) {
        this.forceResponseEncoding = forceResponseEncoding;
    }

    public boolean isForceResponseEncoding() {
        return this.forceResponseEncoding;
    }

    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String encoding = this.getEncoding();
        if (encoding != null) {
            if (this.isForceRequestEncoding() || request.getCharacterEncoding() == null) {
                request.setCharacterEncoding(encoding);
            }

            if (this.isForceResponseEncoding()) {
                response.setCharacterEncoding(encoding);
            }
        }

        filterChain.doFilter(request, response);
    }
}
```

配置如下：
```xml
<filter>
	<filter-name>encodingFilter</filter-name>
	<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
	<init-param>
		<param-name>encoding</param-name>
		<param-value>UTF-8</param-value>
	</init-param>
	<init-param>
		<param-name>forceEncoding</param-name>
		<param-value>true</param-value>
	</init-param>
</filter>
<filter-mapping>
	<filter-name>encodingFilter</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```

[关于URL编码](http://www.ruanyifeng.com/blog/2010/02/url_encoding.html)