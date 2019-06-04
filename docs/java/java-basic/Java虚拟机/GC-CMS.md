# CMS垃圾收集器
1. CMS垃圾收集器注重STW时间
2. 使用算法，基于 标记-清除 算法
3. 只会回收老年代和永久带（1.8开始为元空间，需要设置CMSClassUnloadingEnabled，不会收集新生代
4. 新生代可选：Parallel New/Serial 
5. CMS是一种预处理垃圾回收期，不能等到老年代空间耗尽时回收，需要在内存耗尽前完成回收操作，否则会导致并发回收失败
6. CMS老年代回收阈值为92%
7. CMS不会移动对象以保证空闲空间的连续性

## 缺点
1. 无法处理浮动垃圾，可能出现"Concurrent Mode Failure"失败而导致另一次full gc
2. 收集时会有大量空间碎片产生。空间碎片过多时会出现老年代还有很大的空余空间，但无法找到足够的连续空间来分配当前对象，
   不得不提现出发一次full gc。为了解决这个问题，CMS提供了一个参数：-XX:+UseCMSCompactAtFullCollection 参数，用于
   在CMS收集器顶不住要进行FULL GC时开启内存碎片的合并整理过程，这个参数默认时开启的。内存整理的过程是无法并发的，空间
   碎片的问题没有了，但是停顿时间不得不变长。还提供了一个参数：-XX:+CMSFullGCBeforeCompaction，这个参数是用于设置
   执行多少次不压缩的Full GC，跟着来一次带压缩的（默认为0，每次进入Full GC时都进行碎片整理）
3. CMS收集器对CPU非常敏感。在并发阶段，虽然不会导致用户线程停顿，但是会因为占用一部分线程或CPU资产而导致应用变慢，
   总吞吐量降低。


## GC模式
1. background
   执行正常的CMS GC的步骤
2. foreground
   发生的场景如业务线程请求分配内存，但是内存不够了，于是可能触发一次cms gc，这个过程中，整个业务线程是不可用的

## 触发条件
1. background
   - 应用主动请求Full GC，且开启ExplicitGCInvokesConcurrent直接触发
   - 没有设置 UseCMSInitiatingOccupancyOnly
       - 统计开启，统计的CMS完成时间小于CMS剩余空间被填满的时间，则触发
       - 统计不可用，老年代大于 -XX:CMSInitiatingOccupancyFraction 时触发
   - 设置了 UseCMSInitiatingOccupancyOnly
       - 根据指定老年代的判断逻辑 should_concurrent_collect，true则触发
       - 根据增量模式收集是否失败，incremental_collection_will_fail，true则触发
       - 根据元空间判断逻辑，should_concurrent_collect，true则触发
2. foreground
   - 在eden区为对象或担保对象分配内存失败，即发生 promotion failed 和 concurrent mode failure
   
## 执行步骤

| 步骤 | 说明 | STW |
| --- | ---   | --- |
| 初始标记 | 标记GC ROOT能直接关连到的对象 | 是 |
| 并发标记 | 由前阶段标记过的对象出发，所有可到达的对象都在本阶段中标记 | 否 |
| 并发预清理 | 标记从新生代晋升的对象、新分配到老年代的对象以及在并发标记阶段修改了的对象 | 否 |
| 可被中止的预清理 | 执行一些预清理，以减少remark阶段造成的STW时间 | 否 |
| 重新标记 | 重新扫描堆中的对象，进行可达性分析，标记活着的对象 | 是 |
| 并发清理 | 回收在对象图中不可达对象 | 否 |
| 并发重置 | 做一些收尾工作，以便下一次GC | 否 |

## jdk1.8 CMS实例
1. strings占有引用，无法回收，最终导致CMS GC。
```
public static void main(String[] args) {
    List<String> strings = new ArrayList<>();
    for (int i = 0; i < 60000000; i++) {
        strings.add("test cms");
    }

    try {
        Thread.sleep(1000000000);
        Thread.yield();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

}
```
2. JVM参数
-XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+PrintGC -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:/gc-cms.log

3. 解释
```
1.[GC (CMS Initial Mark) [1 CMS-initial-mark: 457329K(477660K)] 457329K(692636K), 0.0003395 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
执行初始标记，老年代使用=457329K(old区总大小=477660K)时触发了标记，整个堆使用=457329K(整个堆空间大小=692636K) ，STW=0.0003395 secs。   
ps：CMS初始化标记阶段(需要stop the world),这个阶段标记的是由根(root)可直达的对象(也就是root之下第一层对象)，标记期间整个应用线程会停止。

2. [CMS-concurrent-mark-start]
2. [CMS-concurrent-mark: 0.255/0.255 secs] [Times: user=0.61 sys=0.00, real=0.25 secs] 
执行CMS-concurrent-mark(并发标记)    
0.255/0.255 secs分别是占用CPU的时间和墙钟时间此处应用正常运行，STW=0。
ps：开始并发标记阶段，之前被停止的应用线程会重新启动；从初始化阶段标记的所有可达的对象(root之下第一层队形)出发标记处第一层对象所引用的对象(root之下第二层、三层等等)。

3. [CMS-concurrent-preclean-start]
3. [CMS-concurrent-preclean: 0.208/0.208 secs] [Times: user=0.20 sys=0.00, real=0.21 secs]
并发预清理阶段花费0.208秒cpu时间和0.208秒时钟时间,此处应用正常运行，STW=0。
ps：预清理也属于并发处理阶段。这个阶段主要并发查找在做并发标记阶段时从年轻代晋升到老年代的对象或老年代新分配的对象(大对象直接进入老年代)或发生变化的线程(mutators)更新的对象，来减少重新标记阶段的工作量

4. [GC (CMS Final Remark) [YG occupancy: 0 K (214976 K)]
当前使用了0K,年轻代大小为214976K
	2019-05-29T16:37:24.268+0800: 1.721: [Rescan (parallel) , 0.0004857 secs]
		在应用暂停后重新并发标记所有存活对象
	2019-05-29T16:37:24.269+0800: 1.722: [weak refs processing, 0.0000175 secs]
		处理弱引用
	2019-05-29T16:37:24.269+0800: 1.722: [class unloading, 0.0002666 secs]
		卸载已不使用的类
	2019-05-29T16:37:24.269+0800: 1.722: [scrub symbol table, 0.0004205 secs]
		清理symbol table
	2019-05-29T16:37:24.270+0800: 1.722: [scrub string table, 0.0001324 secs][1 CMS-remark: 457329K(477660K)] 457329K(692636K), 0.0014273 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
		重新标记，老年代占用 457329K ，总容量 477660K ；整个堆占用 457329K ，总容量 692636K，STW=0.0014273 secs。

5. [CMS-concurrent-sweep-start]
5. [CMS-concurrent-sweep: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
并发清理总共耗时 0.105秒cpu时间和 0.105秒时钟时间,STW=0。ps:开始并发清理所有未标记或已终结的对象

6. [CMS-concurrent-reset-start]
6. [CMS-concurrent-reset: 0.002/0.002 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
并发重置总共耗时 0.002秒cpu时间 0.002秒时钟时间,STW=0。开始并发重置CMS算法内部数据，为下次垃圾回收做准备。

[Times: user=0.00 sys=0.00, real=0.00 secs]
用户态cpu占用user=0.00秒，内核态cpu占用sys=0，真正物理耗时墙钟时间(也包含线程让出CPU给其他线程执行的时间)real=0.00秒
```

## CMS参数

| 参数 | 意义 | 默认值 |
| ---- | ---  | ----- |
| -XX:+UseConcMarkSweepGC | 新生代ParNew,老年代CMS,concurrent mode failure失败则使用serial Old回收老年代 | jdk1.7和1.8默认是Parallel+Parallel Old,1.9是G1 |
| -XX:ConcGCThreads | 定义并发CMS过程运行时的线程数，若为设置会根据ParallelGCThreads来计算 | --- |
| -XX:+DisableExplicitGC | 禁用 System.gc 调用 | false |
| -XX:+ExplicitGCInvokesConcurrent -XX:+ExplicitGCInvokesConcurrentAndUnloadClasses | CMS background模式是否收集元数据 | false |
| -XX:+CMSClassUnloadingEnabled | 开启元数据区收集，若没有设置，一旦永久区耗尽空间也会尝试进行垃圾回收，但是收集不会是并行的，而再进行一次Full GC | -- |
| -XX:+UseCmsInitiatingOccupancyOnly | JVM通过CMSInitiatingOccupancyFraction的值进行每一次CMS收集，禁止HotSpot自行触发CMS GC | false |
| -XX:CMSInitiatingOccupancyFraction | 该值代表老年代堆空间达到多少使用率进行GC | 默认为-1 |
| -XX:+CMSParallelInitialMarkEnabled | 减少Remark阶段暂停的时间，启用并行Remark，如果Remark阶段暂停时间长，可以启动这个参数 | --- |
| XX:CMSMaxAbortablePrecleanTime | 当abortable-preclean阶段执行达到这个时间时才会结束 | --- |
/ -XX:CMSScheduleRemarkEdenSizeThreshold | 即当eden使用达到此值，才会开始abortable-preclean阶段 | 2m |
| -XX:CMSScheduleRemarkEdenPenetration | 控制abortable-preclean阶段什么时候结束执行 | 50% |
| -XX:+UseCMSCompactAtFullCollection | 在FULL GC的时候，对年老代的压缩 | true |
| -XX:CMSFullGCsBeforeCompaction | 代表多少次Full GC后对老年代做压缩操作，默认为0，代表每次都压缩，可以消除碎片 | 默认0 |
| -XX:+CMSScavengeBeforeRemark | 若Remark阶段暂停时间太长，可以启用这个参数，在Remark之前，先做一个yGC | --- |

## 日志查看
| 参数 | 描述 |
| ---  | --- |
| -XX:+PrintGCDetails | --- |
| -XX:+PrintGCCause | 打印原因，1.8默认开启 |
| -XX:+PrintGCTimeStamps | 输出时间是从JVM启动开始的毫秒数 |
| -XX:+PrintGCDateStamps | 输出格式化系统时间 |
| -Xloggc:../logs/gc.log | --- |
| -XX:+HeapDumpOnOutOfMemoryError -XX:+HeapDumpPath=../dump | OOM时输出dump文件 |
| -XX:+PrintFlagsFinal | 控制台输出所有可配置参数的信息 |

