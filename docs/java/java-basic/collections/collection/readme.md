# Collection

## 层次结构

![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/github/java-summary/java-basic/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/%E9%9B%86%E5%90%88/Collections.png)

## ArrayList

### 结构

![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/github/java-summary/java-basic/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/%E9%9B%86%E5%90%88/array.png)

###特点

- 继承于AbstractList，实现了List、RandomAccess接口，动态的 Object 数组
- 初始容量大小为10，扩容因子为1.5（`oldCapacity + (oldCapacity >> 1)`）
- 在指定索引处添加元素，或是容量不足时添加元素，或是删除指定索引处的元素都会对数组进行一次复制，这是引起效率低下的原因
- 实现了RandomAccess接口，支持随机访问，实际上就是通过所有进行快速访问，查询效率高的原因
- 实现了Serializable接口，可以通过序列化进行传输
- 是线程不安全的，可以通过Collections工具类进行包装，也可以使用juc下的CopyOnWriteArrayList
- 可以添加null元素
- 在使用ArrayList的时候最好能指定其容量大小

### 操作

1. add(E e)，尾部插入
   1. 检测是否需要扩容
   2. 将新元素写入序列尾部，并将size+1
2. add(int index, E e)，指定位置插入
   1. 检测是否需要扩容
   2. 通过数组复制，使得擦汗如位置之后的元素都后移一位，空出位置给带插入的元素
   3. 将新元素写入序列尾部
3. remove，删除
   1. 将待删除元素之后的元素都向前移动一位
   2. 将最后一个元素置为空，并将size减1
   3. 清空被删除节点

### 适用场景

适用于频繁查询和频繁更新数据的场景

## LinkedList

### 结构

![](https://xiaozhang-image.oss-cn-shanghai.aliyuncs.com/github/java-summary/java-basic/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/%E9%9B%86%E5%90%88/linkedlist.png)

### 特点

- 实现了List和Queue(Deque)接口，底层是用带有前后指针的节点维护的双端链表。
- 因为是链表，不存在容量不足的情况
- 因为是链表，存在头和尾，所以可以对头和尾进行操作
- 实现了Serializable接口，可以通过序列化进行传输
- 是线程不安全的，多个线程去修改链表结构的时候，需要保证同步
- 允许null值
- 除List外，还实现了栈和队列的操作方法，因此也可以作为栈、队列和双端队列来使用

### 操作

1. get(int index)
   1. 根据获取的位置判断是从头还是从尾开始便利，然后进行链表遍历
2. add(E e)，在队尾插入
   1. 新元素的prev引用指向旧的最后元素
   2. 旧的最后元素的next引用指向新元素
3. add(int index, E e)，指定位置插入
   1. 更新待插入位置的前一个元素next引用和后一个元素的prev引用指向新的元素
   2. 新元素的prev指向指定位置的前一个元素，next指向指定位置的后一个元素
4. remove
   1. 更新待删除位置的后一个元素的prev指向待删除位置的前一个元素
   2. 更新待删除位置的前一个元素的next指向待删除位置的后一个元素
   3. 清空被删除节点

### 适用场景

适用于频繁插入和频繁删除的场景。