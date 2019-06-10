# servlet/filter/listener/interceptor

## servlet
`servlet`是一种运行服务器端的java应用程序，具有独立于平台和协议的特性，并且可以动态的生成web页面，它工作在客户端请求与服务器响应的中间层。

## filter
`filter`是一个可以复用的代码片段，可以用来转换`HTTP`请求、响应和头信息。`Filter`不像`Servlet`，它不能产生一个请求或者响应，它只是修改对某一资源的请求，或者修改从某一的响应。

## listener
监听器，从字面上可以看出`listener`主要用来监听作用。通过`listener`可以监听`web`服务器中某一个执行动作，并根据其要求作出相应的响应。

通俗的语言说就是在`application，session，request`三个对象创建消亡或者往其中添加修改删除属性时自动执行代码的功能组件。

## interceptor
是在面向切面编程的，就是在你的`service`或者一个方法，前调用一个方法，或者在方法后调用一个方法。

比如动态代理就是拦截器的简单实现，在你调用方法前打印出字符串（或者做其它业务逻辑的操作），也可以在你调用方法后打印出字符串，甚至在你抛出异常的时候做业务逻辑的操作。

## 配置位置
`servlet、filter、listener`是配置到`web.xml`中，`interceptor`不配置到`web.xml`中，`struts`的拦截器配置到`struts.xml`中。`spring`的拦截器配置到`spring.xml`中。

# 加载顺序
`web.xml`的加载顺序是：`context- param -> listener -> filter -> servlet`，按照 `FilterMapping`的顺序执行

# 职责

## servlet
1. 读取客户端（浏览器）发送的显式请求数据。包括 `Html` 表单。
2. 读取由客户端（浏览器）发送的隐式请求数据。包括 `Cookies`，媒体类型等。
3. 处理请求数据并生成结果。
4. 发送显式数据到客户端（浏览器），可以为各种格式。
5. 发送隐式数据到客户端（浏览器），包括设置`Cookies`和缓存。

## filter
1. `filter`能够在一个请求到达`servlet`之前预处理用户请求，也可以在离开`servlet`时处理`http`响应
2. 在执行`servlet`之前，首先执行`filter`程序，并为之做一些预处理工作
3. 根据程序需要修改请求和响应
4. 在`servlet`被调用之后截获`servlet`的执行。

用途：
1. 用户授权的 `filter`：负责检查用户请求，根据请求过滤用户非法请求
2. 日志 `filter`：详细记录某些特殊的用户请求
3. 负责解码的`filter`：包括对非标准编码的请求解码，如`org.springframework.web.filter.CharacterEncodingFilter`
4. 黑名单拦截
5. 跨域，如`org.springframework.web.filter.CorsFilter`

## listener
servlet2.4规范中提供了8个 `listener` 接口，可以将其分为三类，分别如下：

1. 与 `servletContext` 有关的 `listner`接口。
   包括：ServletContextListener、ServletContextAttributeListener

2. 与 `HttpSession` 有关的 `Listner` 接口。
   包括：HttpSessionListner、HttpSessionAttributeListener、HttpSessionBindingListener、 HttpSessionActivationListener

3. 与 `ServletRequest` 有关的 `Listener` 接口.
   包括：ServletRequestListner、ServletRequestAttributeListener

# 生命周期
## servlet
1. init() 方法
   在第一次创建`Servlet`的时候会被调用。通常情况下，用户第一次调用对应该 `Servlet` 的`URL`时，`Servlet` 会被创建。但是当服务器启动时，你也可以指定 `Servlet` 被加载。当用户调用 `servlet` 的时候，每个 `servlet` 的实例就会被创建，并且每一个用户的请求都会产生一个新的线程，在适当的时刻交给 `doGet()` 或是 `doPost()` 方法。

2. service() 方法
   是执行实际任务的主要方法。`Servlet` 容器（Web 服务器）调用 `service()` 方法来处理来自客户端的请求，并将格式化的响应返回到客户端。每次服务器接收到一个 `servlet` 请求时，服务器会产生一个新的线程并调用服务。service() 方法由容器调用，且 `service` 方法在适当的时候调用 `doGet、doPost、doPut、doDelete` 等。所以对 `service()` 方法你什么都不需要做，只是根据你接收到的来自客户端的请求类型来重写 `doGet()` 或 `doPost()`。`doGet()` 和 `doPost()` 方法在每次服务请求中是最常用的方法。下面是这两种方法的特征。

3. destroy() 方法
   `destroy()` 方法只在 `servlet` 生命周期结束时被调用一次。

## filter
## 生命周期：
1. 创建对象：在Web应用加载时执行——只执行一次：说明Filter也是单实例多线程的方式运行
2. 初始化操作：创建对象后立即执行——只执行一次
3. 拦截浏览器请求：执行`doFilter()`对象——多次执行
4. 对象释放前：执行清理操作——只执行一次

# 区别
## servlet
`servlet` 流程是短的，url传来之后，就对其进行处理，之后返回或转向到某一自己指定的页面。它主要用来在业务处理之前进行控制。

## filter
`filter` 流程是线程性的，`url`传来之后，检查之后，可保持原来的流程继续向下执行，被下一个`filter`, `servlet`接收等，而 `servlet` 处理之后，不会继续向下传递。

`filter` 功能可用来保持流程继续按照原来的方式进行下去，或者主导流程，而`servlet`的功能主要用来主导流程。可以将 `filter` 看成是 `servlet`的一个补充（擦屁股的）。

`Filter`可认为是`Servlet`的一种"变种"，它主要用于对用户请求进行预处理，也可以对`HttpServletResponse`进行后处理，是个典型的处理链。

它与`Servlet`的区别在于：它不能直接向用户生成响应。

完整的流程是：`Filter`对用户请求进行预处理，接着将请求交给`Servlet`进行处理并生成响应，最后`Filter`再对服务器响应进行后处理。

## 匹配规则
当一个请求发送到`servlet`容器的时候，容器先会将请求的`url`减去当前应用上下文的路径作为`servlet`的映射`url`，比如我访问的是http://localhost/test/index.html（我的应用上下文是test），

容器会将`http://localhost/test`去掉，将剩下的`/index.html`部分拿来做`servlet`的映射匹配，也就是拿这剩下的部分与`web.xml`中配置的`servlet`的`url-pattern`进行匹配。

注意：这个映射匹配过程是有一定的规则的，而且每次匹配最终都只匹配一个 `servlet`。（这一点和`filter`不同）

`servlet` 匹配规则：当一個`servlet`匹配成功后就不会在往下去匹配了
精确路径的匹配：
例子：比如`servletA` 的`url-pattern`为 `/test`，`servletB`的`url-pattern`为 `/*` ，这个时候，如果我访问的`url`为`http://localhost/test` ，这个时候容器就会先 进行精确 路径匹配，发现`/test`正好被`servletA`精确匹配，那么就去调用`servletA`，也不会去理会其他的`servlet`了。

最长路径的匹配：
例子：`servletA`的`url-pattern`为`/test/*`，而`servletB`的`url-pattern`为`/test/a/*`，此时访问`http://localhost/test/a`时，容器会选择路径最长的`servlet`来匹配，也就是这里的`servletB`。

扩展匹配：
如果`url`最后一段包含扩展，容器将会根据扩展选择合适的`servlet`。
例子：`servletA`的`url-pattern`：`*.action`

`filter`匹配规则
1. 通过URL地址
精确匹配：在`url-pattern`标签中指定一个具体的URL地址，其中不使用任何的通配符
模糊匹配：在`url-pattern`标签中指定一个带有通配符的URL地址
	<1>前缀匹配：`URL`地址前面确定，后面使用通配符，例如：`/happy/*`
	<2>后缀匹配：`URL`地址后面确定，前面使用通配符，例如：`*.jsp，*.jpg`
注意：不能在`URL`地址中间使用通配符，例如：`/happy/*.jsp`是不允许的

2. 映射`Servlet`

## 区别 
`servlet,filter` 都是针对 `url` 之类的，而 `listener`是针对对象的操作的，如 `session` 的创建，`session.setAttribute` 的发生，在这样的事件发生时做一些事情。

可用来进行：`Spring`整合`Struts`,为`Struts`的`action`注入属性，`web`应用定时任务的实现，在线人数的统计等

## interceptor
`interceptor` 拦截器，类似于`filter`,不过在`struts.xml`中配置，不是在`web.xml`,并且不是针对`URL`的，而是针对`action`,当页面提交`action`时，

进行过滤操作，相当于`struts1.x`提供的`plug-in`机制，可以看作，前者是`struts1.x`自带的`filter`,而`interceptor` 是`struts2` 提供的`filter`。

与`filter`不同点：
1. 不在`web.xml`中配置，而是在`struts.xml`中完成配置，与`action`在一起
2. 可由`action`自己指定用哪个`interceptor` 来在接收之前做事 

## struts2中的过滤器和拦截器的区别与联系
1. 拦截器是 `Struts2` 提供的，而过滤器是由 `Servlet` 标准提供的
2. 拦截器拦截目标 `Action` 的目标方法，而过滤器针对各种 `web` 资源
3. 拦截器在 `struts.xml` 中配置，而过滤器在 `web.xml`文件中配置
4. 拦截器使用拦截器栈组织在一起，而过滤器是根据被拦截的资源联系在一起，由他们在配置文件中的位置决定了先后执行顺序
5. 拦截器是基于`java`反射机制的，而过滤器是基于函数回调的。
6. 过滤器依赖与`servlet`容器，而拦截器不依赖与`servlet`容器。
7. 拦截器只能对`Action`请求起作用，而过滤器则可以对几乎所有请求起作用。
8. 拦截器可以访问`Action`上下文、值栈里的对象，而过滤器不能。
9. 在`Action`的生命周期中，拦截器可以多次调用，而过滤器只能在容器初始化时被调用一次。

