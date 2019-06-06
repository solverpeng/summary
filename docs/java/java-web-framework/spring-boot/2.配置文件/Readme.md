# Spring Boot配置文件
Spring Boot使用一个全局的配置文件，配置文件名是固定的：application.properties/application.yml

## 作用
修改Spring Boot自动配置的默认值。

## yaml语法
### 基本语法
1. `key: value`，键值对的方式，中间有一个空格。
2. 以空格的缩进来控制层级关系。同一列数据是同一个层级。
3. 属性和值是大小写敏感的。

### 值的写法
1. 普通字面量：`key: 普通的值（数字、字符串、布尔值）`，字符串默认不用加单引号或者双引号。
    - 双引号不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思
    - 单引号会转义特殊字符，特殊字符最终只是一个普通的字符串数据
    ```yaml
    name: "tom \n jerry" # 输出： tom 换行 jerry
    name: 'tom \n jerry' # 输出： tom \n jerry
    ```
2. 对象和Map：`key: value`，来表明对象属性和值的关系
    ```yaml
    person:
        last-name: zhangsan
        age: 180
    ```
    行内写法
    ```yaml
    person: {lastName: zhangsan,age: 180}
    ```

3. 数组（List/Set）：用`-`表示数组中的一个元素
    ```yaml
    pets:
        - cat
        - dog
        - pig
    ```
    行内写法
    ```yaml
    pets: [cat,dog,pig]
    ```

## 配置文件值注入
1. 配置文件

   ```yaml
   person:
       lastName: hello
       age: 18
       boss: false
       birth: 2017/12/12
       maps: {k1: v1,k2: 12}
       lists:
         - lisi
         - zhaoliu
       dog:
         name: 小狗
         age: 12
   ```

2. Java类

   ```java
   @Component
   @ConfigurationProperties(prefix = "person")
   public class Person {
       private String lastName;
       private Integer age;
       private Boolean boss;
       private Date birth;
   
       private Map<String,Object> maps;
       private List<Object> lists;
       private Dog dog;
   }
   ```

   - @ConfigurationProperties：告诉Spring Boot将本类中所有属性和配置文件中相关配置进行绑定。prefix用来指定和配置文件中那个属性进行一一映射。
   - @Compent：这个组件必须为容器中的组件

3. 导入配置提示依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-configuration-processor</artifactId>
       <optional>true</optional>
   </dependency>
   ```

   导入spring-boot-configuration-processor这个依赖后，编写配置文件的时候就会自动提示。



### @Value

1. 通过具体指定的方式获取配置文件中的值
2. 支持字面量、${key}的方式从环境变量/配置文件中获取值、#{spEL表达式}



###  @ConfigurationProperties

1. 批量注入配置文件中的属性

2. 松散绑定

   ```yaml
   - person.firstName
   - person.first-name
   - person.first_name
   - PERSON_FIRST_NAME 系统属性推荐使用这种方式
   ```

3. JSR303校验

   ```java
   @Component
   @ConfigurationProperties(prefix = "person")
   @Validated
   public class Person {
       @Max(100)
       private Integer age;
   }
   ```

4. 支持复杂类型封装




### @Value和@ConfigurationProperties对比

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |



### @PropertySource
加载指定位置配置文件，和现有配置形成互补。
```java
@Component
@ConfigurationProperties(prefix = "person")
@PropertySource("classpath:person.yml")
public class Person {
}
```

### @ImportResource
1. 导入Spring配置文件，让其生效。

2. 需要标注在一个配置类上

   ```java
   @ImportResource("classpath:beans.xml")
   @SpringBootApplication
   public class SpringbootConfigApplication {
   }
   ```

Spring不推荐使用这种方式，推荐使用全注解的方式来向容器中添加组件。
1. @Configuration 指明当前类是一个配置类，替代基于xml的配置方式
2. @Bean 给容器中添加一个组件，替代基于xml的`<bean>`的方式

```java
@Configuration
public class Beans {
    // 方法的返回值添加到容器中，id为方法名
    @Bean
    public HelloService helloService() {
        return new HelloService();
    }
}
```

### 配置文件占位符
1. 随机数

   ```
   ${random.value}、${random.int}、${random.long}
   ${random.int(10)}、${random.int[1024,65536]}
   ```
2. 通过占位符获取之前配置的值，如果没有也可以指定默认值

   ```
   last-name: zhangsan${random.uuid}
   name: ${person.last-name:tom}_dog
   ```
3. application.properties的优先级要比application.yml高。



## Profile
### 多Profile文件
编写主配置文件的时候，文件名可以是：application-{profile}.properties/yml。默认使用application.properties文件。

### yml支持多文档块支持
```yaml
server:
  port: 8081
spring:
  profiles:
    active: prod

---
server:
  port: 8083
spring:
  profiles: dev


---
server:
  port: 8084
spring:
  profiles: prod  #指定属于哪个环境
```

### 激活指定的profile
1. 在配置文件中指定  spring.profiles.active=dev
2. 命令行：
   java -jar spring-boot-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev；
   可以直接在测试的时候，配置传入命令行参数
3. 虚拟机参数；-Dspring.profiles.active=dev

## 配置文件加载位置
springboot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件

* file:./config/
* file:./
* classpath:/config/
* classpath:/

优先级由高到底，高优先级的配置会覆盖低优先级的配置；SpringBoot会从这四个位置全部加载主配置文件；互补配置；

还可以通过spring.config.location来改变默认的配置文件位置

项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；
指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；

java -jar spring-boot-config-0.0.1-SNAPSHOT.jar --spring.config.location=G:/application.properties

## 配置加载顺序
SpringBoot也可以从以下位置加载配置； 优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置

1. 命令行参数，所有命令都可以在命令行上指定

   java -jar spring-boot-config-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc
   多个配置用空格分开，--配置项=值
2. 由jar包外向jar包内进行寻找，优先加载带profile

   jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件
   
   jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件
3. 加载不带profile

   jar包外部的application.properties或application.yml(不带spring.profile)配置文件
   
   jar包内部的application.properties或application.yml(不带spring.profile)配置文件