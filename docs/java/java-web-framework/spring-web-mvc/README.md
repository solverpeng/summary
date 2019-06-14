# Spring Web MVC

`Spring Web MVC` 是基于 Servlet API 构建的原始Web框架，包含在 `Spring` 框架中，通常也被称为 `Spring MVC`。



## DispatcherServlet

1. 初始化方式
2. 上下文层次结构
3. 特殊的BEAN类型
4. WEB MVC配置
5. Servlet 配置
6. 请求处理
7. 拦截器
8. 异常解析
9. 错误页面
10. 视图解析
	* 视图处理
	* 重定向
	* 转发
11. Locale
12. 主题
13. Multipart 解析器
14. 日志

## 过滤器 Filters

- 表单数据
- 转发头
- Shallow ETag
- CORS

## 带注解的控制器

1. 声明
   * AOP代理

2. Request Mapping

   * URI patterns

   * Pattern Comparison

   * 后缀匹配

   * Suffix Match and RFD

   * 消费者媒体类型

   * 生产者媒体类型

   * 请求参数、请求头

   * HTTP HEAD, OPTIONS

   * 自定义注解

   * Explicit Registrations

3. 处理方法

    - 方法参数
    - 返回值
    - 类型转换
    - Matrix Variables
    - @RequestParam
    - @RequestHeader
    - @CookieValue
    - @ModelAttribute
    - @SessionAttributes
    - @SessionAttribute
    - @RequestAttribute
    - Redirect Attributes
    - Flash Attributes
    - Multipart
    - @RequestBody
    - HttpEntity
    - @ResponseBody
    - ResponseEntity
    - Jackson JSON

4. 模型
5. 数据绑定 DataBinder
6. 异常
   - 方法参数
   - 返回值
   - REST API 异常
7. Controller Advice



## URI 链接

- UriComponents
- UriBuilder
- URI Encoding
- Relative Servlet Requests
- Links to Controllers
- Links in Views

## 异步请求

1. DeferredResult
2. Callable
3. Processing
   - 异常处理
   - 拦截器
   - Compared to WebFlux
4. HTTP Streaming
   - Objects
   - SSE
   - Raw Data
5. Reactive Types
6. Disconnects
7. Configuration

## CORS跨域

1. Introduction
2. 处理
3. @CrossOrigin
4. 全局配置
   - Java配置
   - XML 配置
5. CORS Filter

## Web安全

## HTTP缓存

1. CacheControl
2. Controllers
3. 静态资源
4. ETag Filter

## 视图技术

1. Thymeleaf
2. FreeMarker
   - 视图配置
   - FreeMarker配置
   - 表单处理
3. Script Views
   - 要求
   - Script Templates
4. JSP and JSTL
   - 视图解析
   - JSPs versus JSTL
   - Spring’s JSP Tag Library
   - Spring’s form tag library
5. Tiles
   - 依赖
   - 配置
6. RSS and Atom
7. PDF and Excel
8. Jackson
   - Jackson-based JSON MVC Views
   - Jackson-based XML Views
9. XML Marshalling
10. XSLT Views

## MVC配置

1. 启用MVC配置
2. MVC 配置 API
3. 类型转换
4. 验证
5. 拦截器
6. Content Types
7. Message Converters
8. View Controllers
9. View Resolvers
10. 静态资源
11. 默认Servlet
12. 路径匹配
13. Advanced  Java 配置
14. Advanced XML 配置

## HTTP/2



# 测试

