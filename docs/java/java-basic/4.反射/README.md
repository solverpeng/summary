# 反射

##  为什么要使用反射

需要在运行时才得知并使用编译时完全未知的类，创建其对象，改变其属性，调用其方法。

## 什么是反射？

允许程序在运行时，借助Reflection API取得任何类的内部信息，并直接操纵其属性和方法。

## 类加载的过程？

当程序主动使用某个类时，该类还未被加载到内存，系统会通过以下三个步骤来加载类。

1. 类的加载：将.class文件读入到内存，并为之创建一个Class对象。由类加载器完成。

2. 类的连接：将类的二进制文件合并到JRE中。

3. 类的初始化：由JVM进行类的初始化。


## 类加载器分类

### 系统类加载器
```
ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
```
### 扩展类加载器
```
ClassLoader parent = ClassLoader.getSystemClassLoader().getParent();
```
### 引导类加载器：负责加载核心类库，无法直接获取
```
ClassLoader parent = ClassLoader.getSystemClassLoader().getParent().getParent();
```

## 类加载的时机

1. 源文件程序只有在需要使用该程序代码时，才会被加载。

2. 类的代码只有在初次使用时才会被加载。

   - 通常发生于创建第一个对象的时候

   - 访问static域和static对象时也会被加载。

   - 创建对象引用不加载类。

   - 可以归结为：访问static成员时，类会被初始化，因为构造器也是隐式的静态方法。因此更准确的讲，类在任何static成员访问时加载。

## 获取Class对象的四种方式
```java
Class clazz = Person.class;
```
```java
Person person = new Person();
Class<? extends Person> clazz = person.getClass();
```
```java
Class<?> clazz = Class.forName("com.nucsoft.refletciton.Person");
```
```java
String className = "com.nucsoft.jdbc.Person";
ClassLoader classLoader = Person.class.getClassLoader();
Class<?> clazz = classLoader.loadClass(className);
```

## 通过反射创建实例

1. 默认是通过无参构造器来创建的
```java
String className = "com.nucsoft.jdbc.Person";
Class<?> clazz = Class.forName(className);
Object newInstance = clazz.newInstance();
```
2. 也可以通过有参构造器来创建
```java
Class<?> personClass = Class.forName("com.nucsoft.refletciton.Person");
Constructor<?> constructor = personClass.getConstructor(int.class, String.class);
Object obj = constructor.newInstance(2, "wangwu");
```

## 通过反射获取属性，改变属性

1. getField()方法只能获取public修饰的属性

2. getDeclaredField()方法能获取任意修饰符修饰的属性对象(Field对象)。但是在修改私有属性的时候，需要更改访问权限。
```java
Class<?> personClass = Class.forName("com.nucsoft.refletciton.Person");
Field personName = personClass.getDeclaredField("personName");
personName.setAccessible(true);
personName.set(person, "zhangsan");
```

## 通过反射获取方法，调用方法

1. getMethod() 方法只能获取public修饰的方法

2. getDeclaredMethod() 方法可以任意修饰符修饰的方法对象(Method对象)。但是在调用非公有方法时，需要设置访问权限。且通过method.invoke(obj, args)来调用、
```java
Method sayMethod = personClass.getDeclaredMethod("say");
sayMethod.setAccessible(true);
sayMethod.invoke(person);
```

## 获取父类泛型
```java
Class clazz = Person.class; 
Type type = clazz.getGenericSuperClass(); 
ParameterizedType types = (ParameterizedType)type; 
for(Type t : types) { 
	System.out.println(t); 
}
```

## 静态代理
```java
public interface Factory { void produce(); }
public class NikeFactory implements Factory{

    @Override
    public void produce() {
        System.out.println("生产Nike衣服.....");
    }

}
public class NikeProxy implements Factory{
    private NikeFactory nikeFactroy = null;
    
    public NikeProxy(NikeFactory nikeFactory) {
        this.nikeFactroy = nikeFactory;
    }

    @Override
    public void produce() {
        nikeFactroy.produce();
    }

}
public static void main(String[] args) {
    NikeFactory nikeFactory = new NikeFactory();
    NikeProxy nikeProxy = new NikeProxy(nikeFactory);
    nikeProxy.produce();
}
```

## 动态代理
```java
public interface Factory { void produce(); }
public class NikeFactory implements Factory{

    @Override
    public void produce() {
        System.out.println("生产Nike衣服.....");
    }

}
public class MyInvocationHandler implements InvocationHandler {
    Object obj = null;
    
    public Object getProxy(Object obj) {
        this.obj = obj;
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        return method.invoke(obj, args);
    }
    
}
public class DynamicProxy {
    public static void main(String[] args) {
        NikeFactory nikeFactory = new NikeFactory();
        MyInvocationHandler myInvocationHandler = new MyInvocationHandler();
        Object obj = myInvocationHandler.getProxy(nikeFactory);
        Factory factory = (Factory) obj;
        factory.produce();
    }
}
```

## AOP
```java
public interface Human { void walk(); void fly(); }
public class SuperMan implements Human {

    @Override
    public void walk() {
        System.out.println("走走走走啊走！");
    }

    @Override
    public void fly() {
        System.out.println("I belive i can fly...");
    }

}
public class MyInvocationHandler implements InvocationHandler {
    Object obj = null;
    public Object getProxy(Object obj) {
        this.obj = obj;
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), this);
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        HumanUtil.before();
        Object returnValue = method.invoke(obj, args);
        HumanUtil.after();
        return returnValue;
    }

}
public class HumanUtil {
    public static void before() {
        System.out.println("HumanUtil's before...");
    }
    
    public static void after() {
        System.out.println("HumanUtil's after...");
    }
}
public class AopTest {
    public static void main(String[] args) {
        SuperMan superMan = new SuperMan();
        MyInvocationHandler myInvocationHandler = new MyInvocationHandler();
        Object proxy = myInvocationHandler.getProxy(superMan);
        Human human = (Human)proxy;
        human.walk();
        System.out.println("--------------------");
        human.fly();
    }
}
```