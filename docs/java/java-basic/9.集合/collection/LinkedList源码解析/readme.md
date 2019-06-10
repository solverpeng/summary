# LinkedList源码解析

## 官方介绍

1. 和ArrayList类似，LinkedList是List接口和Deque（双端队列）的双端链表的实现，ArrayList是List接口可变长度数组的实现。实现了所有列表的操作，并且允许null值。
2. 列表中索引的操作将从开始或尾部遍历列表，以更接近指定的索引为准。
3.  LinkedList不是同步的，存在线程安全问题。
4. iterator和listIterator方法返回的迭代器是"fail-fast":在迭代器创建之后，如果从结构上对列表进行修改，除非通过迭代器自身的 remove 或 add 方法，其他任何时间任何方式的修改，迭代器都将抛出 ConcurrentModificationException。

## 数据结构

Node类是LinkedList中的静态内部类，LinkedList就是通过Node来存储集合中的元素。

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

说明

1. next：指向后一个节点的指针
2. prev：指向前一个节点的指针

## 属性

```java
/**
 * 实际元素的个数
 */
transient int size = 0;

/**
 * 头节点
 * Invariant: (first == null && last == null) || (first.prev == null && first.item != null)
 */
transient Node<E> first;

/**
 * 尾节点
 * Invariant: (first == null && last == null) || (last.next == null && last.item != null)
 */
transient Node<E> last;
```

## 构造器

### LinkedList()

