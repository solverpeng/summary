# URL匹配规则
## 规则说明

1. `servlet`容器中的匹配规则既不是简单的通配，也不是正则表达式，而是特定的规则。所以不要用通配符或者正则表达式的匹配规则来看待`servlet`的`url-pattern`。
2. `Servlet` 2.5开始，一个servlet可以使用多个`url-pattern`规则，`<servlet-mapping>`标签声明了与该`servlet`相应的匹配规则，每个`<url-pattern>`标签代表1个匹配规则。
3. 当`servlet`容器接收到浏览器发起的一个`url`请求后，容器会用`url`减去当前应用的上下文路径，以剩余的字符串作为`servlet`映射，假如`url`是`http://localhost:8080/appDemo/index.html`，其应用上下文是`appDemo`，容器会将`http://localhost:8080/appDemo`去掉，用剩下的`/index.html`部分拿来做`servlet`的映射匹配。
4. `url-pattern`映射匹配过程是有优先顺序的。
5. 而且当有一个`servlet`匹配成功以后，就不会去理会剩下的`servlet`了。

## 四种匹配规则

1. 精确匹配
   - `url-pattern`配置内容和`url`完全精确匹配。
   - 如`url-pattern`配置为`<url-pattern>/user/add</pattern>`，会匹配`/user/add`请求，`/user/add/`是非法请求。
2. 路径匹配
   - 以`/`字符开头，以`/*`结尾的字符串用于路径匹配。
   - 如`url-pattern`配置为`<url-pattern>/user/*</pattern>`，如下请求都会被匹配。
     - `/user/add.action`
     - `/user/update.action`
     - `/user/list`
     - `/user/list?param=v1`
3. 扩展名匹配
   - 以`*.`开头的字符串被用于扩展名匹配
   - 如`url-pattern`配置为`<url-pattern>*.action</url-pattern>`，如下请求会被匹配。
     - `/user/add.action`
     - `/order.action`
4. 默认匹配
   - `url-pattern`配置为`/`。

## 匹配顺序

1. 精确匹配
2. 路径匹配，先匹配最长路径，再匹配最短路径
3. 扩展名匹配
4. 默认匹配

## 非法匹配

1. 路径匹配和扩展名匹配无法同时设置
   - 如`<url-pattern>/user/*.action</url-pattern>`是非法的
   - `<url-pattern>/aa/*/bb</url-pattern>`是精确匹配，合法
2. `"/*"`和`"/"`含义并不相同
   - `"/*"`属于路径匹配，并且可以匹配所有request。路径匹配的优先级仅次于精确匹配，所以`"/*"`会覆盖所有的扩展名匹配。一般只用于`filter`的`url-pattern`。
   - `/`是`servlet`中特殊的匹配模式，切该模式有且仅有一个实例，优先级最低，不会覆盖其他任何`url-pattern`，只是会替换`servlet`容器的内建`default servlet`，该模式同样会匹配所有`request`。可能会拦截`/appDemo/user/add`格式的请求，但是不会拦截`/appDemo/user/user.jsp`或`/appDemo/index.jsp`，因为在`Tomcat`的web.xml中配置了`.jsp`的`<url-pattern>`。
   - 都会拦截静态资源的加载。

## 作用对象
1. Servlet
2. Filter，一般设置为`/*`。