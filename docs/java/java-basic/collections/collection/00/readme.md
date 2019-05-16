# ArrayList源码解析

## 属性

```java
/**
 * 默认初始化容量
 */
private static final int DEFAULT_CAPACITY = 10;

/**
 * 空数组，如果传入容量为0时使用
 */
private static final Object[] EMPTY_ELEMENTDATA = {};

/**
 * 空数组，传入容量时使用，添加第一个元素的时候会重新初始为默认容量大小
 */
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * 存储元素的数组
 */
transient Object[] elementData; // non-private to simplify nested class access

/**
 * 集合中元素的个数
 */
private int size;
```

属性说明

1. DEFAULT_CAPACITY：默认容量为10，也即通过new ArrayList()创建时的默认容量
2. EMPTY_ELEMENTDATA：空的数组，通过new ArrayList(0)创建时用的是这个空数组
3. DEFAULTCAPACITY_EMPTY_ELEMENTDATA：空数组，通过new ArrayList()创建时使用这个空数组，添加第一个元素的时候会将这个空数组初始化为容量10
4. elementData：真正存放元素的地方，使用`transient`是为了不序列化这个字段。
5. size：真正存储元素的个数，而不是elementData数组的长度。

