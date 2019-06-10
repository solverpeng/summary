# Spring Boot集成Swagger2

## 添加依赖

```xml
<!--
springfox-swagger2依然是依赖OSA规范文档，也就是一个描述API的json文件，而这个组件的功能就是帮助我们自动生成这个json文件
-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.7.0</version>
</dependency>
<!--
springfox-swagger-ui就是将这个json文件解析出来，用一种更友好的方式呈现出来
-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.7.0</version>
</dependency>
```

## Spring Boot中Swagger2配置类

```java
@Configuration
@EnableSwagger2
public class Swagger2Config {
	@Bean
	public Docket api() {
    	return new Docket(DocumentationType.SWAGGER_2)
            /*
            文档配置
            配置内容包括：文档标题、文档描述、联系、版本、服务条款URL、协议等
             */
            .apiInfo(apiInfo())
            /*
            对swagger公开端点进行细粒度的控制
            选择那些 paths 和 apis 会生成 document，具体由apis()和paths()控制
             */
            .select()
                /*
                 api控制
                 apis()方法允许入参Predicate类型，RequestHandlerSelectors任一选项返回值即为Predicate类型，包含如下选项：
                 RequestHandlerSelectors#any、
                 RequestHandlerSelectors#none、
                 RequestHandlerSelectors#basePackage、
                 RequestHandlerSelectors#withMethodAnnotation、
                 RequestHandlerSelectors#withClassAnnotation
                 */
                .apis(RequestHandlerSelectors.basePackage("com.jianlc"))
                /*
                路径控制
                允许入参Predicate类型，PathSelectors任一选项返回值即为Predicate类型，包含如下选项：
                PathSelectors#any、
                PathSelectors#none、
                PathSelectors#regex、
                PathSelectors#ant
                 */
                .paths(PathSelectors.any())
            .build();
	}

	private ApiInfo apiInfo() {
    	return new ApiInfoBuilder()
            //文档标题
            .title("API接口文档标题")
            //文档描述
            .description("API接口文档描述")
            //联系
            .contact(new Contact("张云鹏", "", ""))
            .build();
	}
    
}
```

## Spring Boot中使用Swagger2

```java
@RestController
@RequestMapping("/hello")
//用在类上，说明该类的作用
@Api(value = "Hello World")
public class HelloSwagger2 {

    //用在方法上，说明方法的作用，标注在具体请求上，value和notes的作用差不多，都是对请求进行说明；
    @ApiOperation(value = "hello", notes = "hello swagger2")
    @GetMapping("/")
    public String hello() {
        return "success";
    }
}
```

启动服务器，访问应用下：`/swagger-ui.html`即可访问到对应的UI界面；访问应用下：`/v2/api-docs`可以访问到生成的swagger json文件，可以集成到别的平台中。

## Restful风格

```java
@RestController
@RequestMapping("/user")
@Api(value = "用户管理")
public class UserController {
    /**
     * userMap
     */
    private static Map<Long, User> userMap = Collections.synchronizedMap(new HashMap<>());


    @ApiOperation(value = "获取用户列表", notes = "获取所有用户信息")
    @GetMapping("/")
    public List<User> userList() {
        return new ArrayList<>(userMap.values());
    }

    @ApiOperation(value="新增用户", notes="根据User对象创建用户")
    @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
    @PostMapping("/")
    public String addUser(@ModelAttribute User user) {
        userMap.put(user.getId(), user);
        return "success";
    }

    @ApiOperation(value="根据id获取用户", notes="根据url的id来获取用户详细信息")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long")
    @ApiResponses({
            @ApiResponse(code = 3001, message = "这就是自定义异常")
    })
    @GetMapping(value = "/{id}")
    public User getUser(@PathVariable Long id) {
        return userMap.get(id);
    }

    @ApiOperation(value="更新用户", notes="根据url的id来指定更新对象，并根据传过来的user信息来更新用户详细信息")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long"),
            @ApiImplicitParam(name = "inputUser", value = "用户详细实体user", required = true, dataType = "User")
    })
    @PutMapping(value = "/{id}")
    public String updateUser(@PathVariable Long id, @ModelAttribute User inputUser) {
        User user = userMap.get(id);
        user.setUserName(inputUser.getUserName());
        user.setAge(inputUser.getAge());
        userMap.put(id, user);
        return "success";
    }

    @ApiOperation(value="删除用户", notes="根据url的id来指定删除对象")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long")
    @DeleteMapping("/{id}")
    public String deleteUser(@PathVariable Long id) {
        userMap.remove(id);
        return "success";
    }
}
```

```java
public class User {
    @ApiModelProperty(value = "用户id")
    private Long id;
    @ApiModelProperty(value = "用户姓名")
    private String userName;
    @ApiModelProperty(value = "年龄")
    private Integer age;
    @ApiModelProperty(value = "性别")
    private String sex;
    @ApiModelProperty(value = "是否有车")
    private Boolean hasCar;
    @ApiModelProperty(value = "存款")
    private BigDecimal bankSavings;
}
```

## 分组管理

项目中存在多个模块时，swagger2也为我们提供了友好的使用方式，具体如下：

```java
@Bean
public Docket userApi() {
    return new Docket(DocumentationType.SWAGGER_2)
            .groupName("demo-user")
            .apiInfo(v1ApiInfo())
            .select()
                .apis(RequestHandlerSelectors.basePackage("com.jianlc."))
                .paths(PathSelectors.ant("/user/**"))
            .build()
            ;
}

@Bean
public Docket orderApi() {
    return new Docket(DocumentationType.SWAGGER_2)
            .groupName("demo-order")
            .apiInfo(v2ApiInfo())
            .select()
            	.apis(RequestHandlerSelectors.any())
            	.paths(PathSelectors.ant("/order/**"))
            .build()
            ;
}
```

在 SpringbootSwagger2 配置文件中，根据不同的模块配置不同的Decket，进一步是通过 Decket#groupName 来控制的，这里需要说明的一点是，组名只是一个标识，并不会起任何拦截作用，不同组的拦截还是通过 apis()和paths() 进行拦截的。在访问生成的swagger2的api json文件时，需要在链接后添加【key=group，value=组名】的请求参数，才可以访问到对应的文件，如上面的例子，访问地址如下：

<http://localhost:8081/v2/api-docs?group=demo-user>

<http://localhost:8081/v2/api-docs?group=demo-order>



## 多版本管理

为了适应当前目前存在的多版本api，这里提供两种不同形式的版本

> ├─java
> │ └─com
> │ └─solverpeng
> │ ├─bean
> │ ├─config
> │ ├─controller
> │ │ ├─v1
> │ │ └─v2
> │ └─result
> └─resources
> ├─static
> └─templates

第一种，如上面目录结构，不同版本是通过包来管理的，来看看Swagger2如何管理这样目录的文档的

```java
@Bean
public Docket userApiV1() {
    return new Docket(DocumentationType.SWAGGER_2)
            .groupName("demo-user-v1")
            .apiInfo(v1ApiInfo())
            .select()
            .apis(RequestHandlerSelectors.basePackage("com.solverpeng.v1"))
            .paths(PathSelectors.ant("/user/**"))
            .build()
            ;
}

@Bean
public Docket userApiV2() {
    return new Docket(DocumentationType.SWAGGER_2)
            .groupName("demo-user-v2")
            .apiInfo(v2ApiInfo())
            .select()
            .apis(RequestHandlerSelectors.basePackage("com.solverpeng.v2"))
            .paths(PathSelectors.ant("/user/**"))
            .build()
            ;
}
```

配置文件如上，对应swagger2生成的json 文件地址如下：

<http://localhost:8081/v2/api-docs?group=demo-user-v1>

<http://localhost:8081/v2/api-docs?group=demo-user-v2>



第二种，多版本不是通过包来划分的，而是通过细粒度请求级别请求的映射值来划分的，如下：

```java

@RestController
@RequestMapping("/order")
@Api(description = "订单管理")
public class OrderController {

    private List<Order> orderList = new ArrayList<>();

    @ApiOperation(value = "订单列表", notes = "获取所有订单")
    @GetMapping(value = "v1/orderList")
    public List<Order> orderListV1() {
        orderList.add(new Order(100L, UUID.randomUUID().toString(), new Date()));
        orderList.add(new Order(101L, UUID.randomUUID().toString(), new Date()));
        return orderList;
    }

    @ApiOperation(value = "订单列表", notes = "获取所有订单")
    @GetMapping(value = "v2/orderList")
    public List<Order> orderListV2() {
        orderList.add(new Order(100L, UUID.randomUUID().toString(), new Date()));
        orderList.add(new Order(101L, UUID.randomUUID().toString(), new Date()));
        return orderList;
    }

}
```

配置文件如下：

```java
@Bean
public Docket orderApiV1() {
    return new Docket(DocumentationType.SWAGGER_2)
            .groupName("demo-order-v1")
            .apiInfo(v2ApiInfo())
            .select()
            .apis(RequestHandlerSelectors.any())
            .paths(PathSelectors.ant("/v1/order/**"))
            .build()
            ;
}

@Bean
public Docket orderApiV2() {
    return new Docket(DocumentationType.SWAGGER_2)
            .groupName("demo-order-v2")
            .apiInfo(v2ApiInfo())
            .select()
            .apis(RequestHandlerSelectors.any())
            .paths(PathSelectors.ant("/v2/order/**"))
            .build()
            ;
}
```



**对比两种不同版本的管理，可以看到，本质上还是通过 apis()和paths() 来控制的，在使用的过程中可能会有更复杂的多版本情况，结合这两个方法，我们也能灵活应对。**



## 规范定义

这里给出组名的规范：项目名-模块名-版本号，如 nb-user-v1。



## swagger2常用annoations

| Name                                                         | Description                  |
| :----------------------------------------------------------- | :--------------------------- |
| [@Api](https://github.com/swagger-api/swagger-core/wiki/Annotations#api) | 描述类/接口的主要用途        |
| [@ApiImplicitParam](https://github.com/swagger-api/swagger-core/wiki/Annotations#apiimplicitparam-apiimplicitparams) | 描述方法的参数               |
| [@ApiImplicitParams](https://github.com/swagger-api/swagger-core/wiki/Annotations#apiimplicitparam-apiimplicitparams) | 描述方法的参数(Multi-Params) |
| [@ApiModelProperty](https://github.com/swagger-api/swagger-core/wiki/Annotations#apimodelproperty) | 添加和操作模型属性的数据     |
| [@ApiOperation](https://github.com/swagger-api/swagger-core/wiki/Annotations#apioperation) | 描述方法用途                 |
| [@ApiParam](https://github.com/swagger-api/swagger-core/wiki/Annotations#apiparam) | 请求属性                     |
| [@ApiResponse](https://github.com/swagger-api/swagger-core/wiki/Annotations#apiresponses-apiresponse) | 响应配置                     |
| [@ApiResponses](https://github.com/swagger-api/swagger-core/wiki/Annotations#apiresponses-apiresponse) | 响应集配置                   |

