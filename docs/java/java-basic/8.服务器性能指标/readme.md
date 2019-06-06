# 服务器性能指标-负载
在UNIX系统中，系统负载是对当前CPU工作量的度量，被定义为特定时间间隔内运行队列中的平均线程数。load average 表示机器一段时间内的平均load。这个值越低越好。负载过高会导致机器无法处理其他请求及操作，甚至导致死机。

Linux的负载高，主要是由于CPU使用、内存使用、IO消耗三部分构成。任意一项使用过多，都将导致服务器负载的急剧攀升。

## 查看机器负载
在Linux机器上，有多个命令都可以查看机器的负载信息。其中包括uptime、top、w等。

### uptime
`uptime` 命令能够打印系统总共运行了多长时间和系统的平均负载。
uptime命令可以显示的信息显示依次为：现在时间、系统已经运行了多长时间、目前有多少登陆用户、系统在过去的1分钟、5分钟和15分钟内的平均负载。
```
[root@iz2ze88p52tbimsqsq3canz ~]# uptime
 11:09:01 up 157 days, 57 min,  2 users,  load average: 0.00, 0.01, 0.05
```
这行信息的后半部分，显示"load average"，它的意思是"系统的平均负荷"，里面有三个数字，我们可以从中判断系统负荷是大还是小。
这三个数字的意思分别是1分钟、5分钟、15分钟内系统的平均负荷。我们一般表示为load1、load5、load15。
 
### w命令
w命令的主要功能其实是显示目前登入系统的用户信息。
w命令还可以显示：当前时间，系统启动到现在的时间，登录用户的数目，系统在最近1分钟、5分钟和15分钟的平均负载。
然后是每个用户的各项数据，项目显示顺序如下：登录帐号、终端名称、远 程主机名、登录时间、空闲时间、JCPU、PCPU、当前正在运行进程的命令行。
 
### top命令
top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。
```
 top - 11:12:58 up 157 days,  1:01,  2 users,  load average: 0.00, 0.01, 0.05
 Tasks:  71 total,   1 running,  69 sleeping,   1 stopped,   0 zombie
 %Cpu(s):  0.3 us,  0.0 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
 KiB Mem :  1883492 total,   141916 free,   425060 used,  1316516 buff/cache
 KiB Swap:        0 total,        0 free,        0 used.  1257916 avail Mem
 
   PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 13824 root       0 -20  127408  12096   9284 S  0.3  0.6  99:51.53 AliYunDun
     1 root      20   0   51488   3656   2448 S  0.0  0.2  21:47.55 systemd
```

## 机器正常负载范围
最好根据自己机器的实际情况，建立一个指标的基线（如近一个月的平均值），只要日常的load在基线上下范围内不太大都可以接收，如果差距太多可能就要人为介入检查了。

阮一峰在自己的博客中有过以下建议：
> 当系统负荷持续大于0.7，你必须开始调查了，问题出在哪里，防止情况恶化。
  当系统负荷持续大于1.0，你必须动手寻找解决办法，把这个值降下来。
  当系统负荷达到5.0，就表明你的系统有很严重的问题，长时间没有响应，或者接近死机了。你不应该让系统达到这个值。

以上指标都是基于单CPU的，但是现在很多电脑都是多核的。所以，对一般的系统来说，是根据cpu数量去判断系统是否已经过载（Over Load）的。
如果我们认为0.7算是单核机器负载的安全线的话，那么四核机器的负载最好保持在3(4*0.7 = 2.8)以下。

## 如何降低负载
导致负载高的原因可能很复杂，有可能是硬件问题也可能是软件问题。

如果是硬件问题，那么说明机器性能确实就不行了，那么解决起来很简单，直接换机器就可以了。前面我们提过，CPU使用、内存使用、IO消耗都可能导致负载高。

如果是软件问题，有可能由于Java中的某些线程被长时间占用、大量内存持续占用等导致。建议从以下几个方面排查代码问题：
1. 是否有内存泄露导致频繁GC 
2. 是否有死锁发生 
3. 是否有大字段的读写 
4. 会不会是数据库操作导致的，排查SQL语句问题。

这里还有个建议，如果发现线上机器Load飙高，可以考虑先把堆栈内存dump下来后，进行重启，暂时解决问题，然后再考虑回滚和排查问题。

## Java Web应用Load飙高排查思路
1. 使用uptime查看当前load，发现load飙高。
2. 使用top命令，查看占用CPU较高的进程ID。
```
➜  ~ top

PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
1893 admin     20   0 7127m 2.6g  38m S 181.7 32.6  10:20.26 java
```
发现PID为1893的进程占用CPU 181%。而且是一个Java进程，基本断定是软件问题。

3. 使用 top命令，查看具体是哪个线程占用率较高
```
➜  ~ top -Hp 1893
PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
4519 admin     20   0 7127m 2.6g  38m R 18.6 32.6   0:40.11 java
```

4. 使用printf命令查看这个线程的16进制
```
➜  ~ printf %x 4519
11a7
```
5. 使用`jstack`命令查看当前线程正在执行的方法
```
➜  ~ jstack 1893 |grep -A 200 11a7
"thread-5" #500 daemon prio=10 os_prio=0 tid=0x00007f632314a800 nid=0x11a2 runnable [0x000000005442a000]
java.lang.Thread.State: RUNNABLE
at sun.misc.URLClassPath$Loader.findResource(URLClassPath.java:684)
at sun.misc.URLClassPath.findResource(URLClassPath.java:188)
at java.net.URLClassLoader$2.run(URLClassLoader.java:569)
at java.net.URLClassLoader$2.run(URLClassLoader.java:567)
at java.security.AccessController.doPrivileged(Native Method)
at java.net.URLClassLoader.findResource(URLClassLoader.java:566)
at org.hibernate.validator.internal.xml.ValidationXmlParser.getInputStreamForPath(ValidationXmlParser.java:248)
at com.solverpeng.test.util.BeanValidator.validate(BeanValidator.java:30)
```
从上面的线程的栈日志中，可以发现，当前占用CPU较高的线程正在执行我代码的com.solverpeng.test.util.BeanValidator.validate(BeanValidator.java:30)类。
那么就可以去排查这个类是否用法有问题了。

6. 还可以使用jstat来查看GC情况，看看是否有频繁FGC，然后再使用jmap来dump内存，查看是否存在内存泄露。

# 服务器性能指标-CPU利用率分析及问题排查
Linux的负载高，主要是由于CPU使用、内存使用、IO消耗三部分构成。任意一项使用过多，都将导致服务器负载的急剧攀升。
本文就来分析其中的第二项，CPU的利用率。主要涉及CPU利用率的定义、查看CPU利用率方式、CPU利用率飙高排查思路等。

## CPU利用率
CPU利用率，又称CPU使用率。顾名思义，CPU利用率是来描述CPU的使用情况的，表明了一段时间内CPU被占用的情况。
使用率越高，说明你的机器在这个时间上运行了很多程序，反之较少。使用率的高低与你的CPU强弱有直接关系。

很多人都知道，现在我们用到操作系统，无论是Windows、Linux还是MacOS等其实都是多用户多任务分时操作系统。
使用这些操作系统的用户是可以“同时”干多件事的，这已经是日常习惯了，并没觉得有什么特别。 
但是实际上，对于单CPU的计算机来说，在CPU中，同一时间是只能干一件事儿的。为了看起来像是“同时干多件事”，
分时操作系统是把CPU的时间划分成长短基本相同的时间区间,即"时间片"，通过操作系统的管理，把这些时间片依次轮流地分配给各个用户使用。 
如果某个作业在时间片结束之前,整个任务还没有完成，那么该作业就被暂停下来,放弃CPU，等待下一轮循环再继续做.此时CPU又分配给另一个作业去使用。 
由于计算机的处理速度很快，只要时间片的间隔取得适当,那么一个用户作业从用完分配给它的一个时间片到获得下一个CPU时间片，中间有所"停顿"，
但用户察觉不出来,好像整个系统全由它"独占"似的。

而我们说到的CPU的占用率，一般指的就是对时间片的占用情况。

使用uptime 、top 、w 等命令可以在Linux查看系统的负载情况。其中，top 命令也可以用来查看CPU的利用率，除此之外，还可以使用vmstat 来查看cpu的利用率。 

## vmstat命令
vmstat命令是最常见的Linux/Unix监控工具，可以展现给定时间间隔的服务器的状态值,包括服务器的CPU使用率，内存使用，虚拟内存交换情况,IO读写情况。
```
procs  -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs   us sy id  wa st
 2  0      0 147604 172224 1143644  0    0     0     1    2    2    0  0 100  0  0
```
从上面的结果中我们可以看到很多信息，我们本文重点关注下cpu部分的指标。
1. %us：用户进程执行时间百分比，us的值比较高时，说明用户进程消耗的CPU时间多，但是如果长期超50%的使用，那么我们就该考虑优化程序算法或者进行加速。
2. %sy：内核系统进程执行时间百分比，sy的值高时，说明系统内核消耗的CPU资源多，这并不是良性表现，我们应该检查原因。
3. %id：空闲时间百分比
4. %wa：IO等待时间百分比，wa的值高时，说明IO等待比较严重，这可能由于磁盘大量作随机访问造成，也有可能磁盘出现瓶颈（块操作）
5. %st：虚拟 CPU 等待实际 CPU 的时间的百分比

一般vmstat工具的使用是通过两个数字参数来完成的，`vmstat 2 2`，第一个参数是采样的时间间隔数，单位是秒，第二个参数是采样的次数。

## top命令
top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。

使用top命令，除了可以查看Load Avg以外，还可以显示CPU利用率信息。


## Java Web应用CPU使用率飙高排查思路
当发现系统的CPU使用率飙高时，首先要定位到是哪个进程占用的CPU较高。一般情况下，对于Java代码来说，导致CPU飙高可能由以下几个原因引起：
1. 内存泄露、导致大量Full GC（如典型的Java 1.7之前的String.subString导致的内存泄露问题） 
2. 代码存在死循环（如典型的多线程场景使用HashMap导致死循环的问题）

基本都是先定位到占用CPU较多的进程和线程，然后通过命令在查看这条线程执行情况。通过分析代码来定位其中的问题。

# 服务器性能指标-内存使用分析及问题排查

# 什么是内存
内存(Memory)也被称为内存储器，其作用是用于暂时存放CPU中的运算数据，以及与硬盘等外部存储器交换的数据。

程序运行时的数据加载,线程并发,I/O缓冲等等,都依赖于内存,可用内存的大小,决定了程序是否能正常运行以及运行的性能。

## 物理内存
物理内存指通过物理内存条而获得的内存空间。即随机存取存储器（random access memory，RAM），是与CPU直接交换数据的内部存储器，也叫主存(内存)。

## 虚拟内存
虚拟内存是计算机系统内存管理的一种技术。它使得应用程序认为它拥有连续可用的内存（一个连续完整的地址空间），
而实际上，它通常是被分隔成多个物理内存碎片，还有部分暂时存储在外部磁盘存储器上，
在需要时进行数据交换（也就是说，当物理内存不足时，可能会借用硬盘空间来充当内存使用）。
与没有使用虚拟内存技术的系统相比，使用这种技术的系统使得大型程序的编写变得更容易，对真正的物理内存（例如RAM）的使用也更有效率。

## Swap分区
Swap分区（即交换区）在系统的物理内存不够用的时候，把硬盘空间中的一部分空间释放出来，以供当前运行的程序使用。
那些被释放的空间可能来自一些很长时间没有什么操作的程序，这些被释放的空间被临时保存到Swap分区中，
等到那些程序要运行时，再从Swap分区中恢复保存的数据到内存中。

# 查看内存使用情况
## free
free命令可以显示Linux系统中空闲的、已用的物理内存，swap分区以及被内核缓冲区内存。
```
[root@iz2ze88p52tbimsqsq3canz ~]# free
              total        used        free      shared  buff/cache   available
Mem:        1883492      417328      149796         356     1316368     1266092
Swap:             0           0           0
```
一共有3行6列数据，行数据的意义如下： 
Mem 行是内存的使用情况。 buffers/cache 行是物理内存的缓存统计情况。 Swap 行是交换空间的使用情况。

### Mem
展示物理内存的整体情况

Total：表示物理内存总大小。
Used ：表示总计分配给缓存（包含buffers 与cache ）使用的数量，但其中可能部分缓存并未实际使用。
Free ：表示未被分配的内存。
Shared：共享内存，一般系统不会用到。
available：可用内存大小。

> total = used + free + buff/cache

### Swap行
Total：Swap内存总大小。
Used ：表示已分配的Swap大小。
Free：表示未被分配的内存。

### buffer与cache的区别
buffers 就是存放要输出到disk（块设备）的数据，缓冲满了一次写，提高IO性能（内存 -> 磁盘）。
cached 就是存放从disk上读出的数据，常用的缓存起来，减少IO（磁盘 -> 内存）。

buffer 和 cache，两者都是RAM中的数据。简单来说，buffer是即将要被写入磁盘的，cache是被从磁盘中读出来的。

### 命令参数
-m 以M为单位显示内存
-g 以G为单位显示内存
-s 2持续的观察内存的状况，每隔2秒打印一次

# Java Web应用内存占用飙高排查思路
JVM以一个进程（Process）的身份运行在Linux系统上。

一般在应用启动时都可以通过JVM参数来设置JVM内存的大小。如果超过这个限制就会抛出异常。所以，我们比较常见的内存占用过高问题，最显著的现象就是抛出各种OutOfMemoryError。

有一种可能导致直接内存，也就是Linux的物理内存过高的情况，就是NIO的使用。NIO引入了一种基于通道与缓冲区的IO方式，
他可以使用Native函数库直接分配堆外内存，然后通过一个存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。
所以，在使用NIO的时候，要特别小心，避免导致机器内存被挤满

导致JVM中内存占用飙高的原因可能有很多。最常见的就是内存泄露。

## 内存泄露排查思路
1. 使用top命令，查看占用内存较高的进程ID。
2. 使用jmap查看内存情况，并分析是否存在内存泄露。
```
jmap -heap 3331：查看java 堆（heap）使用情况

jmap -histo 3331：查看堆内存(histogram)中的对象数量及大小

jmap -histo:live 3331：JVM会先触发gc，然后再统计信息

jmap -dump:format=b,file=heapDump 3331：将内存使用的详细情况输出到文件
```
得到堆dump文件后，可以进行对象分析。如果有大量对象在持续被引用，并没有被释放掉，那就产生了内存泄露，就要结合代码，把不用的对象释放掉。