# 日志
日志门面：[SLF4J](https://www.slf4j.org/)
日志实现：logback、log4j、log4j2、JUL（java.util.logging）

SpringBoot选用 SLF4J 和 logback 实现。

## 使用
使用日志门面 SLF4J 的API
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```
## 如何统一？
![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/github/java-summary/springboot/slf4j.png)
1. 将系统中其他日志框架先排除出去
2. 用中间包来替换原有的日志框架
3. 导入 SLF4J 其他的实现

## SpringBoot日志
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>2.1.5.RELEASE</version>
    <scope>compile</scope>
</dependency>

<!-- spring-boot-starter底层的日志启动器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
    <version>2.1.5.RELEASE</version>
    <scope>compile</scope>
</dependency>

<!-- spring-boot-starter-logging底层的依赖又如下 ,包含了log4j和jul向slf4j转换的中间包-->
<!-- 默认提供了logback 的实现 -->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
    <scope>compile</scope>
</dependency>
<!-- 提供了 log4j 向 slf4j 转换的桥接器 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-to-slf4j</artifactId>
    <version>2.11.2</version>
    <scope>compile</scope>
</dependency>
<!-- 提供了 jul 向 slf4j 转换的桥接器 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>jul-to-slf4j</artifactId>
    <version>1.7.26</version>
    <scope>compile</scope>
</dependency>
```

1. SpringBoot底层也是使用slf4j+logback的方式进行日志记录
2. SpringBoot也把其他的日志都替换成了slf4j
3. 如果要引入其他框架，也不需要对日志进行特别处理，SpringBoot会自动进行转换

> SpringBoot能自动适配所有的日志，而且底层使用slf4j+logback的方式记录日志，引入其他框架的时候，不需要特殊处理。

## SpringBoot日志使用
### 使用默认配置
可以在主配置文件中进行如下配置，具体配置可以参看：LoggingApplicationListener和LoggingSystemProperties。
```yaml
logging:
  level: {com.solverpeng: trace}
  file: myapp.log       
  path: /springboot/log 
  pattern:
    console: %d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n 
    file: %d{yyyy-MM-dd} -> %-5level %logger{50} - %msg%n
```
1. level：为Map类型，key为包名，value为日志级别
2. file：可以指定完整路径，不指定在当前项目下生成
3. path：在指定路径下生成日志文件，使用spring.log作为默认文件，通常不指定file，只指定path，用于归档
4. pattern: 包括控制台和日志文件输出格式，具体格式参见logback日志配置

下面是logging.file和logging.path对比：

| logging.file | logging.path | Example  | Description                        |
| ------------ | ------------ | -------- | ---------------------------------- |
| (none)       | (none)       |          | 只在控制台输出                     |
| 指定文件名   | (none)       | my.log   | 输出日志到my.log文件               |
| (none)       | 指定目录     | /var/log | 输出到指定目录的 spring.log 文件中 |

### 指定配置
在类路径下添加各个日志框架的配置文件，就会覆盖Springboot默认的日志配置。对logback来说，可以添加logback.xml或logback-spring.xml文件。

| logback.xml          | logback-spring.xml                                           |
| -------------------- | ------------------------------------------------------------ |
| 会直接被日志框架识别 | 会先由Springboot进行解析配置，可以使用Springboot的profile功能 |

logback-spring.xml 可以使用`<springProfile name="staging"> `指定某段配置只在某个环境下生效
```xml
<appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
    <layout class="ch.qos.logback.classic.PatternLayout">
        <springProfile name="dev">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ----> [%thread] ---> %-5level %logger{50} - %msg%n</pattern>
        </springProfile>
        <springProfile name="!dev">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ====> [%thread] ===> %-5level %logger{50} - %msg%n</pattern>
        </springProfile>
    </layout>
</appender>
```