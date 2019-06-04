# 防止表单重复提交

表单重复提交的三种情况：

1. 在表单没有到达目标页面前，对请求按钮点击n次。浏览器会将所有点击的请求排成一个队列，先进先出。
2. 在表单提交到达目标页面后，刷新目标页面。
3. 在表单提交后到达目标页面后，点击后退，再次提交。

解决办法：

使用`token`来解决：表单和`Session`域同时维护一个`token`唯一值。

在加载页面的时候，同时向`session`域中和表单隐藏域中放入相同的`token`。

```xml
<%
    String tokenStr = System.currentTimeMillis() + "-" + session.getId();
    session.setAttribute("token", tokenStr);
%>
<form action="/Session/SessionServlet" method="post">
    <input type="hidden" name="tokenParam" value="<%=tokenStr %>" />
    <input type="submit" value="submit">
</form>
```

到达目标方法后：

判断表单提交的`token`值是否和`session`域中的值相等，若相等，则删除`session`域中的`token`值，到达目标页面。

若重复提交，则在`seesion`域中找不到对应的`token`值，跳转到错误页面，给出提示。

```java
String tokenParam = req.getParameter("tokenParam");
HttpSession session = req.getSession();
Object token = session.getAttribute("token");
if(tokenParam == null || !tokenParam.equals(token)) {
    System.out.println("重复提交！！");
    return ;
}        
session.removeAttribute("token");
req.getRequestDispatcher("/target.jsp").forward(req, resp);
```

## Struts2

1. 使用 `<s:token/>` 和 `org.apache.struts2.interceptor.TokenInterceptor` 或 `org.apache.struts2.interceptor.TokenSessionStoreInterceptor`来进行防止表单重复提交。

2. `<s:token/>`
   在页面中使用 `Struts2` 标签 `<s:token/>`  会被解析为：

   ```xml
   <input type="hidden" name="struts.token.name" value="token">
   <input type="hidden" name="token" value="NUV93RXS5OQQVWL1Y1BTKHPPFK3L4UAX">
   ```

3. 拦截器的配置
   默认拦截器栈 `defaultStack`，没有配置 `org.apache.struts2.interceptor.TokenInterceptor` 或是 
   `org.apache.struts2.interceptor.TokenSessionStoreInterceptor`需要手动加入。

   ```xml
   <action name="token" class="com.nucsoft.struts.token.TokenAction">
       <interceptor-ref name="token"/>
       <interceptor-ref name="defaultStack"/>
       <result>/success.jsp</result>
       <result name="invalid.token">/form.jsp</result>
   </action>
   <action name="tokenSession" class="com.nucsoft.struts.token.TokenAction" method="tokenSession">
       <interceptor-ref name="tokenSession"/>
       <interceptor-ref name="defaultStack"/>
       <result>/success.jsp</result>
   </action>
   ```

4. org.apache.struts2.interceptor.TokenInterceptor 和 org.apache.struts2.interceptor.TokenSessionStoreInterceptor 的区别
   TokenInterceptor ：需要配置一个 name="invalid.token" 的一个 result，表单重复提交时，给出提示。提示信息会存放到 actionError 中，可以通过标签 s:actionerror 来获取信息。
   也可以在国际化资源文件中对信息实现定制（指定国际化资源文件基名）：
   如：i18n_zh_CN.properties

   ```xml
   struts.messages.invalid.token=\u8bf7\u4e0d\u8981\u91cd\u590d\u63d0\u4ea4\u8868\u5355（请不要重复提交表单）
   ```

   TokenSessionStoreInterceptor ：不需要配置别的 result ，采取是一种静默模式，表单重复提交时，不会给出提示，但是实际上目标方法只会被执行一次。