# Spring MVC测试

Spring MVC有如下测试的办法

- Servlet API Mocks：模拟Servlet API的实现，用于单元测试Contrllers，Filters和其他Web组件。
- TestContent 框架：支持在`JUnit`和`TestNG`测试中加载Spring配置，包括跨测试方法高效缓存已加载的配置，以及支持使用`MockServletContext`加载`WebApplicationContext`。
- Spring MVC Test：一个框架，也称为`MockMvc`，用于通过`DispatcherServlet`测试带注解的控制器，包含全部的Spring MVC基础结构但没有HTTP服务器。
- Client-side REST：`spring-test`提供了一个`MockRestServiceServer`，可以将其用作模拟服务器，用于测试内部使用`RestTemplate`的客户端代码。
- WebTestClient：专为测试`WebFlux`应用程序而构建，但它也可用于通过HTTP连接对任何服务器进行端到端集成测试。是一个非阻塞，被动的客户端，非常适合测试异步和流式方案。