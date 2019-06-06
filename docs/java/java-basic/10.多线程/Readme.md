# 开始



## 创建线程

### 继承Thread

```java
public class ExtendsThread {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        myThread.start();
    }
}

class MyThread extends Thread {
    @Override
    public void run() {
    }
}
```

### 实现Runnable

```java
public class ImplRunnable {
    public static void main(String[] args) {
        MyRunnable myRunnable = new MyRunnable();
        Thread thread = new Thread(myRunnable);
        thread.start();
    }
}

class MyRunnable implements Runnable {
    @Override
    public void run() {
    }
}
```

实现Runnable接口的好处：多个线程可以共享同一个接口实现类的对象，非常适合多个相同的线程来处理同一份资源。

## Thread类

1. void start(): 启动线程，并执行对象的run()方法
2. run(): 线程在被调度时执行的操作
3. String getName(): 返回线程的名称
4. void setName(String name):设置该线程名称
5. static currentThread(): 返回当前线程