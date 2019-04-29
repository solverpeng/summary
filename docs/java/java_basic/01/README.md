# 内部类\final关键字\static关键字\hashcode方法

## 内部类

### 什么是内部类
一个类定义在另一个类或者一个方法里，称为内部类

### 为什么要使用内部类
每个内部类都可以单独的实现一个接口或是继承一个类，以此来达到多重继承的效果。

### 分类
#### 成员内部类
1. 成员内部类可以无条件访问外部类的所有成员属性和成员方法
2. 若成员内部类拥有和外部类相同的成员变量或方法时，在成员内部类内，默认会调用成员内部类的成员。如果要访问外部类的同名成员，需要以如下方式去访问：
    - 外部类.this.成员变量
    - 外部类.this.成员方法
3. 编译后会出现两个 Class 文件，且内部类会持有外部类对象的一个引用
4. 编译后会出现两个 Class 文件

#### 局部内部类
包括方法内部类和作用域内部类。需要注意在内部类中使用到外部传入的参数时，该参数需要被 final 修饰

1. 局部内部类的访问仅限于方法内或者该作用域内
2. 具备内部类就像是方法里面的一个局部变量一样，不能有 public/protected/private/static 修饰
3. 编译后会出现两个 class 文件

#### 匿名内部类
1. 匿名内部类不能有访问修饰符和static修饰符
2. 匿名内部类在编译的时候由系统自动起名为：OuterClass$n.class 其中 n 为正整数
3. 编译后会出现两个 class 文件

#### 静态内部类
1. 静态内部类不依赖于外部类，所以只能访问外部类的static成员变量和方法
2. 编译后会出现两个 class 文件

### 问题
####  为什么局部内部类和匿名内部类只能访问局部 final 变量
在局部内部类或匿名内部类中访问的局部变量，本质上是本地局部变量的复制，出于数据一致性的考虑，需要 final 修饰

1. 若变量的值在编译期间可以确定，则会在局部内部类或匿名内部类中创建一个拷贝
2. 若局部变量的值在编译期间无法确定，则通过构造器传参的方式来对拷贝进行初始化赋值

说明：反编译局部内部类和匿名内部类，会发现存在两个构造器，一个无参的，一个带参的（参数：一个指向外部类的引用，其余就是方法形参）

### 内部类使用场景和好处
每个内部类都能独立继承一个类或实现一个接口，所以无论外部类是否已经继承了某个类或实现了某个接口，对内部类没有影响，使得多继承方案变的完整。
简而言之，内部类的存在使得Java的多继承机制变得更加完善。

### 创建静态内部类对象的一般形式为
外部类类名.内部类类名 xxx = new 外部类类名.内部类类名()

### 创建成员内部类对象的一般形式为
外部类类名.内部类类名 xxx = 外部类对象名.new 内部类类名()

### 参考资料
[Java内部类详解](http://www.cnblogs.com/dolphin0520/p/3811445.html)


## final关键字
final关键字在Java中是一个保留的关键字，一旦将引用声明为final类型，则不能改变这个引用。

### 基本用法
#### 修饰类
当 final 修饰一个类的时候，该类不能被继承。final 类中的成员变量可以根据需要设置为 final ，但是 final 类中的方法都会被隐式的指定为 final 方法。

#### 修饰方法
用 final 修饰方法时出于两个原因的考虑，第一个是方法锁定，该方法不能被继承。第二个是效率，在低版本的Java中，使用 final 修饰的方法会提高执行效率，
但是在高版本的JAVA中，不需要使用 final 方法进行优化了。

需要注意的是，类的 private 方法会被隐式的指定为 final 方法。

#### 修饰变量
final 可以修饰成员变量和局部变量。若用 final 修饰一个基本数据类型的变量，则其数值一旦在初始化之后便不能再修改了；如果是引用类型的变量，则在对其
初始化之后便不能再指向另一个对象。

当用 final 修饰一个成员变量的时候，成员变量必须在定义时或者构造器中进行初始化赋值。当用 final 修饰一个局部变量的时候，只需要保证在使用前进行了初
始化即可。用 final 修饰的变量，一旦被初始化赋值后，就不能再更改了。

#### 修饰方法参数
final关键字修饰参数，代表了该参数不可更改，在方法中不可更改该参数的值.

在匿名内部类中，为保持参数的一致性，若所传的方法的形参需要在内部类中使用时，需要声明为`final`。

简单理解就是，拷贝引用，为了避免引用值发生改变，例如被外部类的方法修改等，而导致内部类得到的值不一致，于是用`final`来让该引用不可改变。

故如果定义了一个匿名内部类，并且希望它使用一个其外部定义的参数，那么编译器会要求该参数引用是`final`的。

### 面试
#### Demo1
```java
public class Test {
    public static void main(String[] args)  {
        String a = "hello2"; 
        final String b = "hello";
        String d = "hello";
        String c = b + 2; 
        String e = d + 2;
        System.out.println((a == c));
        System.out.println((a == e));
    }
}
```
问：输出什么？
答：true/false。当 final 修饰的是基本数据类型以及 String类型时，如果在编译期间知道它的确切值，会被当做编译期常量使用。即例子中的 c 可以看做：c = "hello2"。
而对于 d 却是需要在运行时进行字符串连接。需要注意的是，需要在编译期间知道 final 变量值。

#### Demo2
```java
public class Test {
    public static void main(String[] args)  {
        String a = "hello2"; 
        final String b = getHello();
        String c = b + 2; 
        System.out.println((a == c));
 
    }
     
    public static String getHello() {
        return "hello";
    }
}
```
因为编译期间不知道 b 的值，所以输入为 false。

#### 被 final 修饰的引用变量指向的对象内容可变么？
答：被 final 修饰的引用变量一旦初始化赋值以后就不能再指向其他的对象，但是它指向的对象的内容是可变的。
```java
public class Test {
    public static void main(String[] args)  {
        final MyClass myClass = new MyClass();
        System.out.println(++myClass.i);
    }
}
 
class MyClass {
    public int i = 0;
}
```
输出结果为1。

## static 关键字
### static 关键字的用途
static 关键字可以用来修饰类的成员变量和成员方法，以及编写 static 静态代码块。
被 static 关键字修饰的成员变量和成员方法不需要依赖对象进行访问，只要类被加载了，
就可以通过类名去访问。简而言之，方便在没有创建对象的情况下进行调用类成员变量和方法。
用 static 修饰的类成员，在内存中只有一份，所有该类对象共有。

### static 修饰的方法
用 static 修饰的方法称为静态方法，可以直接通过类名去调用。静态方法中不能访问类的非静态成员。但是非静态方法中却是可以访问静态成员。

#### static 修饰的变量
用 static 修饰的变量称为静态变量，静态变量和非静态变量的区别是：静态变量被所有对象共享，在内存中只有一个副本，且仅当类初次加载时会被初始化。非静态变量是被各自对象所拥有的，每个对象的非静态变量在内存中存在一个副本，且相互不共享。
static 成员变量按照定义的顺序进行初始化。

#### static 静态代码块
用 static 修饰的代码块称为静态代码块，可以放置在类的各个地方，可以存在多个静态代码块，在类初次加载的时候，会按照 static 块的顺序来执行每个 static 块，且只执行一次。

### 面试
#### static 关键字会改变类中成员的访问权限么？
在 Java 中，类中成员的访问权限只有：private/protected/default/public 四种，static 关键字不会改变类中成员的访问权限。

#### 能通过 this 访问静态成员变量么？
静态变量被所有对象共享，通过 this 可以访问到。

#### static 可以作用于局部变量么？
static 不允许用来修饰局部变量。

#### Demo1
```java
public class Test extends Base{
 
    static{
        System.out.println("test static");
    }
     
    public Test(){
        System.out.println("test constructor");
    }
     
    public static void main(String[] args) {
        new Test();
    }
}
 
class Base{
     
    static{
        System.out.println("base static");
    }
     
    public Base(){
        System.out.println("base constructor");
    }
}
```
输出什么？
答：
```
base static
test static
base constructor
test constructor
```

#### Demo2
```java
public class Test {
    Person person = new Person("Test");
    static{
        System.out.println("test static");
    }
     
    public Test() {
        System.out.println("test constructor");
    }
     
    public static void main(String[] args) {
        new MyClass();
    }
}
 
class Person{
    static{
        System.out.println("person static");
    }
    public Person(String str) {
        System.out.println("person "+str);
    }
}

class MyClass extends Test {
    Person person = new Person("MyClass");
    static{
        System.out.println("myclass static");
    }
     
    public MyClass() {
        System.out.println("myclass constructor");
    }
}
```
输出什么？
答：
```
test static
myclass static
person static
person Test
test constructor
person MyClass
myclass constructor
```

#### 解释
在单一对象的情况下，执行顺序是这样的，按静态成员的先后顺序先执行静态的，然后初始化非静态成员（成员变量、代码块），然后再执行构造器
若存在继承关系，则父类的加载以及初始化在子类之前。

main() 方法时程序入口，在 main() 执行之前，所在类会被加载，不论 main() 方法是否为空。

## hashcode 方法
在 JAVA 中，hashCode() 方法的主要作用是为了配合散列的集合一起使用，这样的散列集合包括：HashSet/HashMap/HashTable。

### 当向散列集合 HashMap 中插入对象时，如何判断该对象是否已经存在？
若调用 equals() 方法来逐一比较，在集合中已经存在很大数据的时候，效率必然会大打折扣。此时 hashCode() 的作用就体现出来了，当集合要 put 新
的对象的时候，先调用这个对象的 hashCode() 方法，得到对应的一个 hashCode 值，实际上在 HashMap 的具体实现中会用一个 table 保存已经存进去的
对象的 hashCode 值，若 table 中没有该 hashCode 值，就可以直接存储，若 table 中存在该 hashCode 值，它就调用 equals() 方法与新元素进行比较，
相同的话就不存储，不同的话就替换已经存在的值，这样调用 equals 方法的次数就大大降低了。
下面这段代码是 java.util.HashMap#put 方法：
```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
```

### 当从散列集合 HashMap 中获取对象时，是根据什么来获取的对象？
先来看一个例子：
```java
public class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Person person = (Person) o;

        if (age != person.age) return false;
        return !(name != null ? !name.equals(person.name) : person.name != null);

    }

}
```
只是重写了 equals() 方法。

```java
public class TestHashCode {

    @Test
    public void test() {
        Person person = new Person("Jack", 12);
        System.out.println(person.hashCode());
        HashMap<Person, String> hashMap = new HashMap<>();
        hashMap.put(person, "a");

        Person jack = new Person("Jack", 12);
        System.out.println(jack.hashCode());

        System.out.println(hashMap.get(jack));
    }
}
```

输出的信息：
```
751872434
926219290
null
```
可以看到，hashCode 值不同，从 hashMap 中获取 jack 为 null。

来看 HashMap 的 get 方法：
```java
public V get(Object key) {
    if (key == null)
        return getForNullKey();
    Entry<K,V> entry = getEntry(key);

    return null == entry ? null : entry.getValue();
}

final Entry<K,V> getEntry(Object key) {
    if (size == 0) {
        return null;
    }

    int hash = (key == null) ? 0 : hash(key);
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```
可以看到，HashMap 的 get() 方法获取值的时候，实际上比较的是 hashCode 值，在存放 hashCode 的 table 中若找不到对应的值，则返回 null。
在上面的例子中，因为 Person 类没有重写 hashCode() 方法，所以 person 和 jack 对象得到了不同的 hashCode 值，返回 null 也就不足为奇。

### hashCode() 的作用
通过上面的两个例子，我们明白了 hashCode() 是配合散列一起使用的，不论是集合中对象的添加还是获取，实际上都与对象的 hashCode() 生成的 
hashCode 值做了相应的比较。也明白了，实际上会有一个 table 来存放生成的 hashCode 值。

### hashCode 值
Java 中的 hashCode() 方法就是根据一定的规则将与对象相关的信息（如对象的存储地址、对象的字段）映射成一个数值，这个数值称为散列值或是哈希码。

### 几个结论
不能直接根据 hashCode 值判断两个对象是否相等，因为不同的对象可能会生成相同的 hashCode 值，虽然概率很小，但是也存在这样的情况。
但是如果两个对象的 hashCode 值不同，这两个对象肯定不相等。判断两个对象是否相等，必须通过 equals() 方法。
也就是说，若两个对象调用 eqauls 方法得到的结果为 true ，则两个对象的 hashCode 值一定相等。
若调用 equals 方法得到的结果为 false，则 hashCode 值一定不相等。
若两个对象的 hashCode 值相等，则 equals 方法得到的结果未知。

### 设计 hashCode() 的初衷
无论何时，对同一对象调用 hashCode() 都应该产生同一个值。
只要 eqauls 方法的比较操作用到的信息没有被修改，那么对这同一对象的调用多次，hashCode() 必须始终如一地返回同一个整数。
```java
public class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Person person = (Person) o;

        if (age != person.age) return false;
        return !(name != null ? !name.equals(person.name) : person.name != null);

    }

    @Override
    public int hashCode() {
        int result = name != null ? name.hashCode() : 0;
        result = 31 * result + age;
        return result;
    }
}
```
hashCode 值依赖于 name 和 age。
```java
@Test
public void test() {
    Person jack = new Person("Jack", 12);
    HashMap<Person, String> hashMap = new HashMap<>();
    hashMap.put(jack, "a");
    jack.setAge(15);

    System.out.println(hashMap.get(jack));
}
```
输出为:null
因为在 put 和 get 之间，改变了 hashCode 值。