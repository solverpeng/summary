# SpEL
Spring 表达式语言，在使用的时候类似于 EL 表达式，但是需要注意的是，SpEL 使用在 Spring Config 文件中。

## 格式
使用 `#{}` 作为界定符，所有在大括号中的字符都将被认为成是 SpEL

## 作用
1.通过 Bean 的 id 对 Bean 进行引用
2.调用方法以及引用对象中的属性
3.计算表达式的值
4.正则表达式的匹配

# SpEL 字面值
1.整数：`<property name="age" value="#{23}"/>`
2.小数：`<property name="salary" value="#{2300.55}"/>`
3.科学计数法：`<property name="salary" value="#{1e4}"/>`
4.字符串：`<property name="empName" value="#{'emp01'}"/>` 或 `<property name='empName' value='#{"emp01"}'/>`
5.布尔值：`<property name="formal" value="#{false}"/>`

# 引用 Bean、属性和方法
## 引用其他 Bean
```xml
<bean class="com.nucsoft.spring.bean.Employee" id="employee">
　　<property name='empName' value='#{"emp01"}'/>
　　<property name="age" value="#{23}"/>
</bean>

<bean class="com.nucsoft.spring.bean.Department" id="department">
　　<property name="deptName" value="#{'dept01'}"/>
　　<property name="employee" value="#{employee}"/>
</bean>
```
## 引用其他 Bean 的属性
```xml
<bean class="com.nucsoft.spring.bean.Employee" id="employee">
　　<property name='empName' value='#{"emp01"}'/>
　　<property name="age" value="#{23}"/>
</bean>

<bean class="com.nucsoft.spring.bean.Department" id="department2" p:deptName="AAAA" p:employee-ref="employee">
</bean>

<bean class="com.nucsoft.spring.bean.Department" id="department">
　　<property name="deptName" value="#{department2.deptName}"/>
　　<property name="employee" value="#{employee}"/>
</bean>
```
控制台输出：
Department{deptName='AAAA', employee=Employee{empName='emp01', age=23}}
注意：引用其他 Bean 的属性是通过 getXxx() 方法来引用的

## 调用方法，支持链式操作
```xml
<bean class="com.nucsoft.spring.bean.Department" id="department">
　　<property name="deptName" value="#{department2.deptName.toString().toLowerCase()}"/>
　　<property name="employee" value="#{employee}"/>
</bean>
```
控制台输出：
Department{deptName='aaaa', employee=Employee{empName='emp01', age=23}}

# SpEL 支持的运算

## 数学运算符：+，-，*，/，%，^
+：
```xml
<bean class="com.nucsoft.spring.bean.Employee" id="employee">
　　<property name='empName' value='#{"emp01"}'/>
　　<property name="age" value="#{23}"/>
　　<property name="salary" value="#{53.32 + 12.23}"/>
</bean>
```
控制台输出：
Employee{empName='emp01', age=23, salary=65.55}
其他运算符使用类似。

## 字符串连接：+
```xml
<property name='empName' value='#{"emp01" +" "+ 12}'/>
```
控制台输出：
Employee{empName='emp01 12', age=23, salary=-41.09}

## 比较运算符：<，>，==，<=，>=，lt，gt，eq，le，ge
```xml
<property name="formal" value="#{100 == 100}"/>
```
控制台输出：
Employee{empName='emp01 12', age=23, salary=-41.09, formal=true}

## 逻辑运算符：and，or，not，|
```xml
<property name="formal" value="#{100 == 100 and 100 gt 80}"/>
```
控制台输出：
Employee{empName='emp01 12', age=23, salary=-41.09, formal=true}
其他几个与之类似。

## if-else 运算符：? exp1 : exp2
```xml
<property name="formal" value="#{100 == 100 ? false : true}"/>
```
控制台输出：
Employee{empName='emp01 12', age=23, salary=-41.09, formal=false}

## 正则表达式：matches
```xml
<property name="matchesEmail" value="#{'a@b.com' matches '/^([a-z0-9_\.-]+)@([\da-z\.-]+)\.([a-z\.]{2,6})'}"/>
```
控制台输出：
isMatchesEmail=false

# 调用静态方法或静态属性。通过 T()， 返回一个类的对象
```xml
<property name="salary" value="#{T(java.lang.Math).PI * 1000}"/>
```
控制台输出：
salary=3141.592653589793