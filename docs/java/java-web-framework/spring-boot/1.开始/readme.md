# 开始
需求：浏览器发送 `hello` 请求，服务器接收请求并处理，响应`Hello World~`。

## 步骤
1. 创建一个maven工程
2. 导入Spring boot依赖
    ```xml
    <!-- 继承默认的Spring Boot 配置 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
    </parent>

    <!-- Add typical dependencies for a web application -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <!-- 将应用打包为一个可执行的Jar文件 -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    ```
3. 编写主程序类，启动Spring boot
    ```java
    @SpringBootApplication
    public class SpringbootApplicationContext {
        public static void main(String[] args) {
            SpringApplication.run(SpringbootApplicationContext.class, args);
        }
    }
    ```
4. 编写 Controller
    ```java
    @Controller
    public class HelloController {

        @ResponseBody
        @RequestMapping("/hello")
        public String hello() {
            return "hello world!";
        }
    }
    ```
5. 可以将应用打成Jar包，使用`java -jar hello-springboot-1.0-SNAPSHOT.jar`执行。

## pom依赖梳理
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.5.RELEASE</version>
</parent>

<!-- 父项目 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.1.5.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
```
`spring-boot-dependencies`管理Spring Boot 应用里面的所有依赖版本。也称之为 Spring Boot的版本仲裁中心。导入依赖默认是不需要写版本，没有在版本仲裁中心管理的依赖仍需要声明版本号。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- 父项目 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starters</artifactId>
    <version>2.1.5.RELEASE</version>
</parent>
```
`spring-boot-starter`：Spring Boot场景启动器，导入了对应模块正常运行所依赖的组件。

Spring Boot将所有的功能场景都抽取出来，做成一个个的starters（启动器），只需要在项目里面引入这些starter相关场景的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器。

## @SpringBootApplication
表示该类是Spring Boot的主配置类。Spring Boot就应该运行这个类的main方法来启动Spring Boot应用。

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM,
				classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    @AliasFor(annotation = EnableAutoConfiguration.class)
	Class<?>[] exclude() default {};

    @AliasFor(annotation = EnableAutoConfiguration.class)
	String[] excludeName() default {};

    @AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};

    @AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
	Class<?>[] scanBasePackageClasses() default {};
}
```

### @SpringBootConfiguration
```java
@Configuration
public @interface SpringBootConfiguration {
}
--------------
@Component
public @interface Configuration {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
}
```
底层其实使用的是`@Component注解`，表明该配置类也是容器中的一个组件。

### @EnableAutoConfiguration
开启自动配置的功能。
```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
    Class<?>[] exclude() default {};
    String[] excludeName() default {};
}
```

```java
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {
}
```
AutoConfigurationPackage用来自动配置包，将主配置类的所在包及下面所有子包里面的所有组件扫描到Spring容器

```java
@Import(AutoConfigurationImportSelector.class)
```
导入自动配置选择器。将所需要导入的组件以全类名的方式返回，这些组件就会被添加到容器中。
会给容器中添加非常多的自动配置类（xxxAutoConfiguration），AutoConfigurationImportSelector就是用来给容器中导入这个场景需要的所有组件，并配置好这些组件。
Spring Boot 在启动的时候会从类路径下`META-INF/spring.factories`获取AutoConfigurationImportSelector指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作。
