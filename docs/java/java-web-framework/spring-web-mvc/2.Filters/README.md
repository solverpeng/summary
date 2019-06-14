# Filters

Spring-web 模块提供了一些有用的`filters`：

- 表单数据
- 字符集编码
- 转发标头
- 浅ETag
- 跨域（CORS）

## 表单数据

浏览器只能通过`HTTP GET`或`HTTP POST`提交表单数据，但非浏览器客户端可以额外使用`HTTP PUT`，`PATCH`和`DELETE`。Servlet API要求`ServletRequest.getParameter*()`方法仅支持`HTTP POST`的表单字段访问。

`spring-web`模块提供`FormContentFilter`来拦截`HTTP PUT`，`PATCH`和`DELETE`请求，其`content-type`为`application/x-www-form-urlencoded`，从请求体读取表单数据，并包装`ServletRequest`，使表单数据通过`ServletRequest.getParameter*()`系列方法可用。



> Spring 版本为5.1.6，在实际测试中发现，不配置该过滤器，对于`HTTP PUT`，`PATCH`和`DELETE`的请求，也能通过`ServletRequest.getParameter*()`获取到表单数据。



## 字符集编码

`Spring-web`模块提供了`CharacterEncodingFilter`来对Http请求和响应字符集编码进行设置。

```java
public class CharacterEncodingFilter extends OncePerRequestFilter {
	@Nullable
	private String encoding;
	private boolean forceRequestEncoding = false;
	private boolean forceResponseEncoding = false;
    
    String encoding = getEncoding();
    if (encoding != null) {
        if (isForceRequestEncoding() || request.getCharacterEncoding() == null) {
            request.setCharacterEncoding(encoding);
        }
        if (isForceResponseEncoding()) {
            response.setCharacterEncoding(encoding);
        }
    }
}
```

通过这三个属性对请求和响应进行编码设置。

需要注意的是，该设置对 @ResponseBody 标注的处理器不起作用。需要设置请求头：Accept 或者在处理器方法处标注 @RequestMapping(produces="application/json;character=utf-8")。



## 转发标头

由于请求会经过像负载均衡器这样的代理，host，port，scheme可能会发生更改，那么host，port，scheme在创建一些资源文件的链接的时候返回给客户端可能是有所不同的。

RFC7239定义了代理可以定义`Forwarded`的Http头来提供原始请求信息。同时还有一些非标准头文件，包括`X-Forwarded-Host`,`X-Forwarded-Port`, `X-Forwarded-Proto`, `X-Forwarded-Ssl`, 和`X-Forwarded-Prefix`。

`ForwardedHeaderFilter`是一个Servlet过滤器，它根据`Forwarded`标头修改请求的主机，端口，然后删除这些标头。

由于应用程序不知道这些标头是由代理还是由恶意客户端添加的，所以需要对转发的标头进行安全考虑。应该将代理配置为删除来自外部的不受信任的转发标头。还可以将`ForwardedHeaderFilter`配置为`removeOnly=true`，在这种情况下，它将删除但不使用标头。



## 浅ETag

`ShallowEtagHeaderFilter`过滤器通过缓存写入响应的内容并从中计算MD5哈希来创建 ETag。当下一次客户端发送请求时，它执行相同的操作，并将计算的值与 `If-None-Match` 请求头进行比较，如果两者相等，则返回304（NOT_MODIFIED）。

此策略可节省网络带宽，但不能节省CPU，因为必须为每个请求计算完整响应。控制器级别的其他策略可以避免计算。参看HTTP缓存。

此过滤器具有`writeWeakETag`参数，该参数将过滤器配置为写入弱ETag，类似于以下内容：W /“02a2d595e6ed9a0b24f027f2b63b134d6”。



> <https://jinnianshilongnian.iteye.com/blog/1608234>



## 跨域(Cors)

`SpringMVC`通过控制器上的注解为CORS配置提供细粒度的支持。然而，当和`Spring Security`一起使用时，我们建议依赖内置的`Corsfilter`，且必须放在`Spring Security`的过滤器链之前。