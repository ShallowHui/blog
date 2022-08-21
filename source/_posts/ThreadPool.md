---
title: 自己动手写一个线程池
date: 2021-11-17 23:56:34
tags: 线程池
categories: Java
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/java.jpg
description: 自己动手写一个简单的线程池Java实现，加深对线程池各个核心参数的理解吧~
---
## 线程池的定义

首先来看一下什么是线程池，根据百度百科：

>线程池是一种多线程处理形式，处理过程中将任务添加到队列，然后在创建线程后自动启动这些任务。

简单来说，线程池可以接受我们提交的任务，然后为每一个任务分配一个线程去完成它。通常我们创建一个异步任务，是开启一个线程：

``` java
new Thread(Runnable r);
```

但这样写会让程序中到处都是创建线程的代码，所以我们需要一个线程池这样的工具类，统一去提交、执行任务。

## 定义线程池相关的接口

既然线程池可以执行我们提交的任务，那给线程池定义一个接口：

``` java
public interface Executor {
    public void execute(Runnable r); // Runnable对象是我们提交的任务，会执行其中的run()方法
}
```

受资源限制，线程池中的线程数量当然是有限的，那当线程池的承载能力满了之后，我们再提交任务时，线程池应该如何处理？最粗暴的做法就是直接丢弃这个任务，但最好可以让用户自己决定如何处理来拒绝这个任务，这叫执行拒绝策略，我们定义一个接口：

``` java
// 处理线程池拒绝提交任务的策略
public interface RejectedExecutionHandler {
    /**
     * 任务提交失败时执行的方法
     * @param executor Executor的实现类
     * @param r 任务对象
     */
    public void rejectedHandle(Executor executor, Runnable r);
}
```

嗯，既然拒绝策略可以自定义，那线程池中，线程的创建也让用户自定义吧，我们定义一个用户自定义创建线程的接口：

``` java
// 线程工厂类，用户可以自定义线程的创建
public interface ThreadFactory {
    public Thread newThread(Runnable r);
}
```

## 实现线程池

### 最简单的线程池

好了，准备工作完成，我们来写一个最简单的线程池，也就是Executor接口的实现类：

``` java
public class ThreadPool implements Executor {
    public void execute(Runnable r) {
        new Thread(Runnable r).start();
    }
}
```

别看它简单，就是直接创建一个线程去执行传入的Runnable对象，没有什么拒绝策略、线程工厂之类的，但JDK中线程池的作者举的例子就是这个，从这个开始构建线程池，一步步添加功能。

### 引入CorePoolSize

正如前面所说，资源是有限的，不可能传入一个任务就创建一个线程去执行。所以我们考虑对线程池中的线程数量进行限制，引入两个参数：

1. CorePoolSize：核心线程数
2. BlockingQueue：任务的等待阻塞队列

引入参数后线程池的代码如下：

``` java
public class ThreadPool implements Executor {

    private final int corePoolSize; // 核心线程数
    private final AtomicInteger workCount = new AtomicInteger(0); // 正在工作的线程数，初始化为零
    private final RejectedExecutionHandler rejectedExecutionHandler; // 拒绝策略
    private final BlockingQueue<Runnable> blockingQueue; // 任务的等待阻塞队列
    private final ThreadFactory threadFactory; // 线程工厂

    // 构造方法
    public ThreadPool(int corePoolSize, RejectedExecutionHandler rejectedExecutionHandler, BlockingQueue<Runnable> blockingQueue, ThreadFactory threadFactory) {
        this.corePoolSize = corePoolSize;
        this.rejectedExecutionHandler = rejectedExecutionHandler;
        this.blockingQueue = blockingQueue;
        this.threadFactory = threadFactory;
    }

    /**
     * 工作线程类，即线程池中具体运行的线程对象是Worker类的实例对象
     */
    private final class Worker implements Runnable {

        private Runnable task; // 工作线程的任务

        public final Thread thread; // 以Worker实例对象自身通过ThreadFactory创建一个Thread，再保存到实例中

        /**
         * Worker类的构造方法
         * @param firstTask 创建工作线程时，设置要运行的第一个任务
         */
        public Worker(Runnable firstTask) {
            this.task = firstTask;
            this.thread = threadFactory.newThread(this);
            workCount.getAndIncrement(); // 工作线程数量加一
        }

        // 执行任务
        public void run() {
            while (true) {
                if (task != null)
                    task.run();
                task = blockingQueue.poll(); // 不断尝试从等待队列中取出新任务
            }
        }
    }
}
```

简单实现RejectedExecutionHandler和ThreadFactory接口：

``` java
public class RejectedImpl implements RejectedExecutionHandler {

    public void rejectedHandle(Executor executor, Runnable r) {
        // 直接输出线程池和任务的信息
        System.out.println("Task: " + r.toString() + " rejected from: " + executor.toString());
    }

}
```

``` java
public class ThreadFactoryImpl implements ThreadFactory {

    public Thread newThread(Runnable r) {
        // 可以设置线程对象的各个属性，这里就直接返回一个线程了
        return new Thread(r);
    }

}
```

**引入这两个参数之后，可以想象一下，一开始线程池中是空的，没有线程在工作。然后每传入一个任务就通过线程工厂创建一个新线程去执行任务，同时workCount加一。当workCount等于corePoolSize时，再传入一个任务，这时就不会去创建新的线程了，而是把这个任务丢进等待队列中，等待工作中的线程执行完当前任务后从队列中取出新任务去执行。如果队列满了，线程池就执行拒绝策略，拒绝这个任务。**

**workCount记录正在工作的线程数，当创建新线程时+1，当后面会讲到的非核心线程因为没有任务执行就销毁时-1，因为有多个线程对这个成员变量进行操作，所以需要使用JUC提供的原子类。同理，多个线程对等待队列进行操作，所以使用线程安全的BlockingQueue。**

按照上面的思路，线程池类中的execute()方法很容易就可以实现，这里就不展示了，可以尝试自己完善线程池类~

重点在下面的弹性扩展。

### 可弹性扩展的线程池

引入上面的参数后，线程池就会逐步创建出corePoolSize个核心线程，并且等待队列中可能有任务在等待。如果恰好这时核心线程都在工作并且等待队列满了，又来了个新任务，那么线程池只能拒绝这个任务了。

考虑这么一个场景：任务在某一段时间正常提交，不是很频繁，线程池的大小足够支撑这种并发量。但突然有一段时间，任务提交非常频繁，提交了非常多的任务，线程池只能丢弃超出其承载能力的任务，这该如何解决？

可以考虑将corePoolSize设置得很大，提高线程池的容量。但这显然有个问题，当提交任务的高峰期过去，就会有若干个核心线程在那“空转”：不断尝试从等待队列中取出新任务，但队列中又没有那么多任务。

为此，特意再引入几个参数：

1. MaxPoolSize：最大工作线程数，指线程池允许的最大工作线程数
2. TimeUnit：时间单位
3. KeepAliveTime：非核心线程的空闲时间，与时间单位一起，就是指非核心线程的最大空闲时间

**maxPoolSize将工作中的线程分为了核心线程和非核心线程：核心线程是常驻内存的，如果当前任务执行完，会不断尝试从等待队列中取出新任务去执行，而非核心线程如果执行完当前任务，没有在空闲时间内从等待队列中取出新任务，就会被销毁。**

**引入maxPoolSize参数后，可以这样理解线程池的最大容量 = maxPoolSize + 等待队列的大小。**

改进后的线程池代码如下：

``` java
public class ThreadPool implements Executor {

    private final int corePoolSize; // 核心线程数（常驻内存）
    private final int maxPoolSize; // 最大工作线程数（核心线程+非核心线程）
    private final AtomicInteger workCount = new AtomicInteger(0); // 正在工作的线程数
    private final RejectedExecutionHandler rejectedExecutionHandler; // 拒绝策略
    private final TimeUnit timeUnit; // 时间单位
    private final long keepAliveTime; // 非核心线程的空闲时间（非核心线程不常驻内存，空闲时间内未能从等待队列中获得任务则结束运行）

    private final BlockingQueue<Runnable> blockingQueue; // 任务的等待阻塞队列
    private final ThreadFactory threadFactory; // 线程工厂

    /**
     * 通过此方法给线程池提交任务
     * @param r 要执行的任务对象
     */
    public void execute(Runnable r) {
        if (workCount.get() < corePoolSize) {
            // 创建核心线程
            Worker coreWorker = new Worker(r, true);
            coreWorker.thread.start();
        } else {
            // 如果任务进入等待队列失败, 判断是否可以创建非核心线程，不能创建则走拒绝策略
            if (!blockingQueue.offer(r)) {
                if (workCount.get() < maxPoolSize) {
                    Worker nonCoreWorker = new Worker(r, false);
                    nonCoreWorker.thread.start();
                } else {
                    rejectedExecutionHandler.rejectedHandle(this, r);
                }
            }
        }
    }

    // 构造方法
    public ThreadPool(int corePoolSize, int maxPoolSize, RejectedExecutionHandler rejectedExecutionHandler,
                      TimeUnit timeUnit, long keepAliveTime, BlockingQueue<Runnable> blockingQueue, ThreadFactory threadFactory) {
        this.corePoolSize = corePoolSize;
        this.maxPoolSize = maxPoolSize;
        this.rejectedExecutionHandler = rejectedExecutionHandler;
        this.timeUnit = timeUnit;
        this.keepAliveTime = keepAliveTime;

        this.blockingQueue = blockingQueue;
        this.threadFactory = threadFactory;
    }

    /**
     * 工作线程类，即线程池中具体运行的线程对象是Worker的实例
     */
    private final class Worker implements Runnable {

        private Runnable task; // 工作线程的任务
        private boolean core;

        public final Thread thread; // 以Worker实例对象自身通过ThreadFactory创建一个Thread，再保存到实例中

        /**
         * Worker的构造方法，可以判断要创建的是不是核心线程
         * @param firstTask 创建工作线程时，设置要运行的第一个任务
         * @param core 判断是否为核心线程
         */
        public Worker(Runnable firstTask, boolean core) {
            this.task = firstTask;
            this.core = core;
            this.thread = threadFactory.newThread(this);

            workCount.getAndIncrement(); // 工作线程数量加一
        }

        // 执行任务
        public void run() {
            if (core)
                runCoreWorker();
            else
                runNonCoreWorker();
        }

        // 核心线程常驻内存
        public void runCoreWorker() {
            while (true) {
                if (task != null)
                    task.run();
                task = blockingQueue.poll();
            }
        }

        // 非核心线程会结束运行
        public void runNonCoreWorker() {
            while (task != null) {
                task.run();
                try {
                    // 要在空闲时间内从队列中取出新任务
                    task = blockingQueue.poll(keepAliveTime, timeUnit);
                } catch (InterruptedException e) {
                    task = null;
                    e.printStackTrace();
                }
            }
            workCount.getAndDecrement(); // 非核心线程结束运行，工作线程数量减一
        }
    }
}
```

**这时，当队列满了，线程池就不是直接丢弃任务了，而是继续创建非核心线程去执行任务，同时workCount继续增加，超过corePoolSize。当workCount等于maxPoolSize时，再提交任务，线程池才执行拒绝策略。这样，当任务的提交量变小时，随着等待队列中的任务一个个被取出来执行完毕，非核心线程会一一销毁，workCount就逐步减小，但workCount是不会小于corePoolSize的，因为核心线程一旦被创建出来就常驻内存。**

### 使用线程池

首先定义一个任务类，再将任务传入线程池中：

``` java
public class Main {

    /**
     * 编写一个Runnable接口的实现类，run方法内定义了要执行的任务
     */
    static class Task implements Runnable {

        public long x;

        public Task(long x) {
            this.x = x;
        }

        public String toString() {
            return "休眠" + x + "秒";
        }

        public void run() {
            try {
                TimeUnit.SECONDS.sleep(x);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("========================================================================>休眠" +
                    x + "秒的任务已由" + Thread.currentThread().getName() + "线程完成");
        }
    }

    public static void main(String[] args) throws Exception {
        RejectedExecutionHandler rejectedExecutionHandler = new RejectedImpl();
        ThreadFactory threadFactory = new ThreadFactoryImpl();
        BlockingQueue<Runnable> blockingQueue = new ArrayBlockingQueue<>(9); // 这里new的队列大小就是线程池中等待队列的大小

        ThreadPool threadPool = new ThreadPool(3, 6, rejectedExecutionHandler, TimeUnit.MILLISECONDS, 100L, blockingQueue, threadFactory);

        // 提交任务
        threadPool.execute(new Task(56L));
        threadPool.execute(new Task(86L));
        threadPool.execute(new Task(76L));
    }
}
```

## 总结

这篇文章讲解了一下线程池的核心参数和基本工作原理，但还不够完善，像JDK中的`ThreadPoolExecutor`线程池还有对线程池状态进行控制的参数。

对于上面可弹性扩展的线程池的完整代码可以参考如下链接：

[https://github.com/ShallowHui/interesting-small-project/tree/master/ThreadPool](https://github.com/ShallowHui/interesting-small-project/tree/master/ThreadPool)