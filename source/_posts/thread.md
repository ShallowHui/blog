---
title: Java多线程
date: 2020-04-27 12:44:45
tags: 并发编程
categories: Java
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/thread.jpg
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/thread.jpg
description: 这篇文章介绍了进程、线程的基本概念，以及如何在Java中进行简单的多线程编程，并且考虑到了并发带来的安全问题。
---
## 前言

随着微处理器技术的飞速发展，个人计算机上的操作系统纷纷改用多任务和分时的设计。多任务就是指操作系统可以同时运行多个应用程序，为此操作系统引入了`进程`的概念。

## 概述

### 程序

程序就是指我们写的代码，在计算机世界内部其实也就是一条条指令，加上程序里要用到的数据，统统被保存在硬盘或者其它外部存储设备里。也就是说程序是静态的代码。

### 进程

硬盘里的程序经过一系列复杂的步骤后，被操作系统从硬盘调到主存中，然后CPU去访问主存把指令一个个取出来执行，这就是计算机的工作原理。在这里，这个程序的一次执行过程就叫做一个进程，换句话说，一个进程就是一个正在执行的程序，即代码是动态的。进程是操作系统资源分配和处理器调度的基本单位，各自拥有独立的代码、内部数据和运行状态。

多任务就是指在一个系统中可以同时运行多个进程，即有多个独立运行的任务，每个任务对应一个进程，每个进程都有自己专用的内存区域，即使是多次启动同一个程序，产生的多个进程也是如此。所谓的同时运行的进程，其实是由操作系统将系统资源分配给各个进程，每个进程在CPU上交替运行，也就是下面要说的并发执行。每个进程占有不同的内存区域，内存消耗很大，这使得系统在不同的应用进程之间切换时开销很大，进程之间的通信速度也很慢。

### 并行和并发

并发执行和并行执行并不相同。

在计算机中，一条指令是有它的指令周期（取址，间址，执行，中断）的，同一时刻，CPU上只能处理一条指令，这样一条条指令依次执行下去，我们叫做串行执行。

那如何同一时刻执行多条指令呢？这往往需要多个处理器的硬件支持，多条指令在多个CPU上同时执行，这就叫做并行执行。

在一个CPU上其实我们也可以做到伪”并行“的效果。这是因为计算机的运行速度非常快，一个指令周期人是根本感受不到的，那么在一个极短的时间段内，这个极短时间段也是人根本感受不到的，多条指令交替执行，这样给人的感觉就是这些指令是”同时“执行的，即所谓的”微观串行，宏观并行“，并发执行就是这样实现的。

+ 并发执行体现在程序上，就是将本来要顺序执行的程序代码，分成一个个相对独立的代码段，每一个代码段都是一个逻辑上相对完整的程序，那一会去执行一下这个代码段里的代码，一会又去执行一下那个代码段里的代码，这样快速交替执行这些代码段，看上去这些代码段就是“同时”执行的，这就实现了并发执行。

+ 在单处理器上，同一时刻始终还是只能执行一条代码，并发执行实质上还是串行执行。

### 线程

**为了减轻系统的负担，引入了线程的概念，将系统资源分配和处理器调度的基本单位分离。进程现在只是资源分配的基本单位，线程是处理器调度的基本单位。一个进程包含一个以上的线程，进程中的所有线程共享该进程的内存区域和数据。**

所谓的线程，其实与进程相似，也是一个执行中的程序，但线程是一个比进程更小的执行单位，有自身的产生、运行、消亡的过程：

![线程的生命周期](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/thread/thread.png)

以往开发的程序，大多是单线程的，即一个程序只有从头到尾顺序执行这一条路径，一般也就是main()方法所在线程，我们把它叫做主线程。然而现实世界中很多的过程都具有多条途径同时运作的特征。比如，我们可以一边喝咖啡，一边听音乐。再比如，一个Web服务器需要同时处理多个客户端的请求。

多线程就是指在同一个进程中同时存在多个执行体，将程序执行过程并发化。一个进程在执行过程中可以产生多个线程，形成多条执行路径。线程不能单独存在，必须存在于进程中，由于同一个进程中的线程是共享同一块内存区域的，所以系统在产生一个线程，或是在各个线程之间切换时，负担要比进程小很多，线程间的通信也快得多。让CPU在同一时间段内执行一个程序中的多个代码段，使工作完成得更有效率，这就是多线程的概念。

+ 要注意并发、多线程、多任务这些概念的区别。多线程是通过并发实现的，并发可以实现多个代码段的同时执行，那多线程就通过并发实现程序的多条执行路径同时执行。多线程和多任务是两个不同的概念。多任务是针对操作系统而言的，表示操作系统可以运行多个应用程序。而多线程是针对一个进程而言的，表示在一个进程内部同时执行多个线程。

### 线程的优先级和调度

在多线程的程序中，每个线程都被赋予了一个执行优先级。优先级决定了线程被CPU执行的优先顺序。优先级高的线程可以在一段时间内获得比优先级低的线程更多的执行时间。

+ Java语言中线程的优先级从低到高以整数1~10表示，共分为10级。主线程的优先级默认为5。

线程的调度就是指在各个线程之间分配CPU资源，多个线程的并发执行实际上是通过一个调度模型来进行的。线程调度的模型有两种：分时模型和抢占模型。

在分时模型中，CPU资源是按时间片分配的，即把CPU的使用时间分成一片一片的。获得CPU使用权的线程，只能在指定的一个时间片内执行程序，一旦这个时间片使用完毕，就必须把CPU使用权让给另一个处于就绪状态的线程。分时模型的调度是没有优先级一说的，所有线程轮流获得CPU的使用权，并且平均分配时间片，每个线程获得相同数量的时间片。

在抢占模型中，优先让优先级高的线程使用CPU，如果优先级相同，则随机选择一个线程执行。一旦一个线程获得了CPU的使用权，这个线程将一直执行下去，直到该线程因为某种原因而不得不让出CPU的使用权。比如，一个正在执行的低优先级线程，碰到一个准备就绪的高优先级线程，就会把CPU使用权让出来，或者线程由于某种原因进入阻塞状态，又或者线程执行完毕。那么，为了让低优先级的线程有机会执行，应该让高优先级的线程时不时地进入“休眠”状态。

**Java语言支持的就是抢占式调度模型**

+ 值得注意的是，线程的调度不是跨平台的，是跟当前的操作系统有关的。在某些系统中，一个线程即使没有执行完，也没有遇到阻塞，一切正常也可能会自动让出CPU的使用权，给其它线程执行的机会。这就可能会让你觉得程序运行结果不符合抢占式调度模型。

## Java实现多线程编程

Java在语言层次上对多线程直接提供支持，顺带一提，Java语言对多线程的支持是建立在调用操作系统提供的API基础之上的，这也就是说为什么线程的调度不是跨平台的。

Java语言实现多线程的方法有两种：一种是继承java.lang包中的Thread类；另一种是用户在自定义的类中实现Runnable接口。

+ 由于Java语言支持多线程，所以只要发现程序中代码其实可以同时工作，那就可以创建一个独立于main()方法所在的主线程之外的新线程来同时进行工作。在一般情况下，如果执行程序的机器是单(核)处理器的话，那么其实程序的执行时间并不会因为并发而减少，只不过程序整体看上去执行效率更高，给人的感觉更好，这一点需要明确。

### 利用Thread类的子类创建线程

Java的基本类库中已定义了Thread这个基本类，其中内置了一组方法。利用这些方法可以去产生一个新的线程、执行一个线程、终止一个线程或让其消亡、查看线程的执行状态。下面是Thread类的一些常用方法：

![Thread类的常用方法](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/thread/threadmethod01.png)

![Thread类的常用方法](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/thread/threadmethod02.png)

构造方法：

![Thread类的构造](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/thread/threadcreate.png)

**还有一个最重要的方法：**

    pubic void run(); //线程要执行的任务

简单来说，就是必须要完成下面两件事：

1. 继承Thread类。

2. 重写run()方法，将线程所要执行的代码写在run()方法内。

+ 线程执行时，是从它的run()方法开始执行的。run()方法是线程执行的起点，就像main()方法是应用程序的执行起点，init()方法是小程序的执行起点一样。但我们在实际激活线程时，一般不直接调用run()方法，而是通过Thread类提供的start()方法来启动线程。

#### 实例

``` java
class myThread extends Thread{                          //继承Thread类
    private String who;                                 //定义一个私有属性
    public String getWho(){
        return who;
    }
    public myThread(String who){
        this.who=who;
    }
    public void run(){                                  //重写覆盖掉Thread类中的run()方法
        for(int i=0;i<5;i++){
            try{         //因为sleep()方法会抛出InterruptedException异常，所以要写在try-catch块里
                sleep((int)(1000*Math.random()));       //1000*Math.random()会让线程随机休眠0~1s
            }
            catch(InterruptedException e){}
            System.out.println(who+"的线程正在执行!");
        }
    }
}
public class test{
    public static void main(String[] args){
        myThread you=new myThread("你");
        myThread she=new myThread("她");
        System.out.println("主线程的优先级为:"+Thread.currentThread().getPriority()); //先获取主线程的线程对象，再获得主线程的优先级
        System.out.println(you.getWho()+"的线程优先级为:"+you.getPriority());
        System.out.println(she.getWho()+"的线程优先级为:"+she.getPriority());
        you.start();                                  //启动线程
        she.start();
        for(int i=0;i<5;i++){
            System.out.println("main()方法所在的主线程正在执行!");
        }
    }
}
```

运行结果：

![继承Thread类](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/thread/Threadclass.png)

可以看出，新创建的线程优先级默认都是5，跟主线程的优先级一样。我们注意到，main()方法里的输出主线程的循环语句是写在最后面的，但它在另外两个线程之前就输出了，前面两条start语句不是将两个线程都启动了吗？这是因为main()方法所在的主线程启动了两个线程之后，是继续往下执行，还是跳到那两个线程中执行，全看这三个线程谁先抢到了CPU的使用权。一般来说是会先往下执行的，因为主线程是早就启动了的，不用再经过线程激活的过程，可以轻易先取得CPU的使用权继续往下执行。不过这里主线程可以先执行完，主要原因是因为另外两个线程激活后就马上休眠了。主线程执行完后，那两个线程的执行就看谁的休眠时间短，谁就可以得到更多的CPU使用时间。

每次运行的结果都可能不同，再运行一遍的结果：

    主线程的优先级为:5
    你的线程优先级为:5
    她的线程优先级为:5
    main()方法所在的主线程正在执行!
    main()方法所在的主线程正在执行!
    main()方法所在的主线程正在执行!
    main()方法所在的主线程正在执行!
    main()方法所在的主线程正在执行!
    她的线程正在执行!
    你的线程正在执行!
    她的线程正在执行!
    她的线程正在执行!
    你的线程正在执行!
    你的线程正在执行!
    她的线程正在执行!
    她的线程正在执行!
    你的线程正在执行!
    你的线程正在执行!

通过让线程休眠的方式，我们看到了Java语言的线程调度模型是抢占式的。但其实去掉休眠语句，并且还把其中一个线程的优先级调到最高，会发现程序运行结果可能不是抢占式的：

``` java
class myThread extends Thread{                          //继承Thread类
    private String who;                                 //定义一个私有属性
    public String getWho(){
        return who;
    }
    public myThread(String who){
        this.who=who;
    }
    public void run(){                                  //重写覆盖掉Thread类中的run()方法
        for(int i=0;i<5;i++){
            System.out.println(who+"的线程正在执行!");
        }
    }
}
public class test{
    public static void main(String[] args){
        myThread you=new myThread("你");
        myThread she=new myThread("她");
        System.out.println("主线程的优先级为:"+Thread.currentThread().getPriority()); //先获取主线程的线程对象，再获得主线程的优先级
        you.setPriority(10);                          //将该线程设置为最高优先级
        System.out.println(you.getWho()+"的线程优先级为:"+you.getPriority());
        System.out.println(she.getWho()+"的线程优先级为:"+she.getPriority());
        you.start();                                  //启动线程
        she.start();
        for(int i=0;i<5;i++){
            System.out.println("main()方法所在的主线程正在执行!");
        }
    }
}
```

运行结果：

    主线程的优先级为:5
    你的线程优先级为:10
    她的线程优先级为:5
    main()方法所在的主线程正在执行!
    main()方法所在的主线程正在执行!
    main()方法所在的主线程正在执行!
    你的线程正在执行!
    你的线程正在执行!
    你的线程正在执行!
    main()方法所在的主线程正在执行!
    main()方法所在的主线程正在执行!
    她的线程正在执行!
    你的线程正在执行!
    她的线程正在执行!
    她的线程正在执行!
    你的线程正在执行!
    她的线程正在执行!
    她的线程正在执行!

我们可以发现，最高优先级的线程在抢到CPU使用权后，并没有按照抢占式调度模型一直执行下去直到线程完成任务，执行完毕。而是会自动让出CPU的使用权，即使线程没有遇到问题。倒是因为没有让其它线程休眠，主线程还没执行完就被高优先级的线程抢走了CPU使用权。

这并不用感到奇怪，前面就提到过，线程的调度是跟当前操作系统有关的。这篇博客的代码是在MacOS上调试完成的，可能MacOS的线程调度模式就是这样的，不用过于纠结这个。可能想深入理解并发编程的原理，还得深入理解计算机操作系统，了解操作系统是如何管理进程，线程的。

### 利用Runnable接口来创建线程

前面介绍了利用Thread类来创建线程，但如果一个类继承了其它的类，由于Java是不支持多继承的，这个类就无法再继承Thread类了。这种情况下，就可以通过实现Runnable接口来创建线程。Runnable接口是Java语言中实现线程的接口，定义在`java.lang`包中，其中只提供了一个run()抽象方法。实质上，Thread类就是实现了Runnable接口，并增加了一组线程的方法，其子类才具有创建线程的功能。

但由于Runnable接口并没有任何对线程支持的方法，所以需要用到Thread类的一个构造方法`Thread(Runnable target)`或者其它构造方法，只要参数中有Runnable实现类对象，我们把Runnable接口实现类的对象称为可运行对象，只需要把这个可运行对象作为参数传递给Thread类的构造方法，就可以创建出一个线程。线程会自动调用接口实现类中的run()方法。

#### 实例

``` java
class myThread implements Runnable{                     //实现Runnable接口
    private String who;                                 //定义一个私有属性
    public String getWho(){
        return who;
    }
    public myThread(String who){
        this.who=who;
    }
    public void run(){                                  //重写覆盖掉run()方法
        for(int i=0;i<5;i++){
            try{
                //由于没有继承Thread类，所以要通过Thread类调用其静态方法
                Thread.sleep((int)(1000*Math.random()));
            }
            catch(InterruptedException e){}
            System.out.println(who+"的线程正在执行!");
        }
    }
}
public class test{
    public static void main(String[] args){
        //先创建可运行对象
        myThread you=new myThread("你");
        myThread she=new myThread("她");
        //再通过可运行对象创建线程对象
        Thread t1=new Thread(you);
        Thread t2=new Thread(she);
        System.out.println("主线程的优先级为:"+Thread.currentThread().getPriority()); //先获取主线程的线程对象，再获得主线程的优先级
        System.out.println(you.getWho()+"的线程优先级为:"+t1.getPriority());
        System.out.println(she.getWho()+"的线程优先级为:"+t2.getPriority());
        t1.start();                                  //启动线程
        t2.start();
        for(int i=0;i<5;i++){
            System.out.println("main()方法所在的主线程正在执行!");
        }
    }
}
```

运行结果：

    主线程的优先级为:5
    你的线程优先级为:5
    她的线程优先级为:5
    main()方法所在的主线程正在执行!
    main()方法所在的主线程正在执行!
    main()方法所在的主线程正在执行!
    main()方法所在的主线程正在执行!
    main()方法所在的主线程正在执行!
    她的线程正在执行!
    她的线程正在执行!
    她的线程正在执行!
    你的线程正在执行!
    她的线程正在执行!
    她的线程正在执行!
    你的线程正在执行!
    你的线程正在执行!
    你的线程正在执行!
    你的线程正在执行!

## 线程间的数据共享

同一进程中的多个线程可以共享同一块内存区域，并可以利用这些共享单元来实现数据的交换，实时通信和必要的同步操作。

### 通过继承Thread类实现数据共享

``` java
class myThread extends Thread{        //继承Thread类
    public myThread(String name){
        super(name);                  //调用父类Thread类的一个构造方法，这样可以给线程对象起名字
    }
    public void run(){
        for(int i=0;i<5;i++){
            test.revoke();            //调用test类的静态方法对共享数据进行改动
            System.out.println(getName()+"执行了revoke()方法：total="+test.getTotal()); //getName()方法获得当前线程的名字
        }
    }
}
public class test{
    private static int total=10;      //定义一个模拟的共享数据
    public static int getTotal(){
        return total;
    }
    public static void revoke(){
        total--;
    }
    public static void main(String[] args){
        myThread t1=new myThread("一号线程");
        myThread t2=new myThread("二号线程");
        t1.start();                    //启动线程
        t2.start();
    }
}
```

运行结果：

    二号线程执行了revoke()方法：total=8
    二号线程执行了revoke()方法：total=7
    一号线程执行了revoke()方法：total=8
    一号线程执行了revoke()方法：total=5
    一号线程执行了revoke()方法：total=4
    二号线程执行了revoke()方法：total=6
    二号线程执行了revoke()方法：total=2
    一号线程执行了revoke()方法：total=3
    二号线程执行了revoke()方法：total=1
    一号线程执行了revoke()方法：total=0

### 通过实现Runnable接口来实现数据共享

前面我们学会了通过创建Runnable接口实现类的可运行对象来创建线程，知道线程执行时会调用可运行对象里的run()方法。那么我们可以用同一个可运行对象创建多个线程，这样一来多个线程就共享同一个可运行对象里的数据了。

``` java
class myThread implements Runnable{
    private int tickes=10;               //定义一个模拟的共享数据
    public void run(){
        while(tickes>0){
            System.out.println(Thread.currentThread().getName()+"卖出了第"+tickes+"张票!");
            tickes--;
        }
    }
}
public class test{
    public static void main(String[] args){
        myThread t=new myThread();              //先创建一个可运行对象
        Thread t1=new Thread(t,"一号窗口");      //再用同一个可运行对象创建多个线程
        Thread t2=new Thread(t,"二号窗口");
        Thread t3=new Thread(t,"三号窗口");
        t1.start();                             //启动线程
        t2.start();
        t3.start();
    }
}
```

运行结果：

    二号窗口卖出了第10张票!
    二号窗口卖出了第9张票!
    三号窗口卖出了第10张票!
    一号窗口卖出了第10张票!
    一号窗口卖出了第6张票!
    三号窗口卖出了第7张票!
    二号窗口卖出了第8张票!
    三号窗口卖出了第4张票!
    一号窗口卖出了第5张票!
    三号窗口卖出了第2张票!
    二号窗口卖出了第3张票!
    一号窗口卖出了第1张票!

通过上面两个不同实现方法的例子，我们可以看出线程间实现了数据共享。但是都存在一个问题，即数据出现了脏读、重读。在第二个例子里，同一张票居然卖出多次，这在现实生活中就是一个很严重的bug了。我们可以这样理解：“二号窗口”线程读到了数据为10，输出10然后将数据改为9，准备把9写入到内存变量中的时候，“三号窗口”线程抢到了CPU的使用权或者“二号窗口“线程进入阻塞状态，这时候“三号窗口”线程去读内存里的变量，就还是没改变之前的10。这样就造成了数据的重读，接下来各个线程按照自己读到的数据去执行run()方法，就可能会出现各种各样的问题。

**这些问题出现的根本原因就是并发的线程共享同一块内存区域造成的，要解决这些问题，就要引入线程间的同步控制。**

## 多线程的同步控制

我们再来看一个共享数据不安全的例子：

``` java
class Mbank{                                  //定义一个银行的模拟类
    private int sum=2000;                     //模拟银行余额作为线程间的共享数据
    public void take(int x){                  //模拟银行取款的方法
        sum-=x;
        System.out.println("sum="+sum);       //余额减去取款数后输出余额
    }
}
class myThread extends Thread{                //继承Thread类
    private static Mbank bank=new Mbank();    //实例化一个唯一的静态对象
    public void run(){                        //重写覆盖run()方法
        for(int i=0;i<5;i++){
            try{
                sleep(100);                   //让线程休眠
            }
            catch(InterruptedException e){}
            bank.take(100);                   //调用Mbank类的静态方法对共享数据进行修改
        }
    }
}
public class test{
    public static void main(String[] args) {
        myThread c1=new myThread();
        myThread c2=new myThread();
        c1.start();                           //启动两个线程
        c2.start();
    }
}
```

运行结果：

    sum=1800
    sum=1800
    sum=1600
    sum=1600
    sum=1400
    sum=1400
    sum=1300
    sum=1300
    sum=1200
    sum=1100

该程序的本意是两个线程分别调用5次Mbank类的静态取款方法， 这样一来，程序运行结果应该是余额递减输出，并且每次减100，最终余额应该为1000。

要解决这样的问题，就必须要对线程进行同步控制。

同步控制要保证，一个线程对数据的操作，不能被其它线程所打断，或修改结果被其它线程的修改结果覆盖掉，即线程间要“互斥”地运行。

**Java中，利用了对象的“互斥锁”机制来实现线程间的互斥操作。“互斥”即一个线程在对一个对象中的共享资源进行访问时，它拿到了这个对象的“锁”，那其它线程就不可以再访问这个对象中被保护起来的共享资源，必须等有“锁”的线程执行完毕，释放掉“锁”。**

Java语言中，使用`synchronized`关键字来标识特定的共享资源，以实现同步限制。这个关键字的功能就是：首先判断对象的互斥锁是否在，若在线程就获得对象的互斥锁，然后就可以访问特定的共享资源，如果不在，线程就进入等待，直到获得锁才能对特定的共享资源进行访问。

+ 专业的说法是，这些共享的数据、资源，称为`临界资源`，而访问这些资源的代码，称为`临界区`代码。

+ 一个普通类的实例化对象，在一个应用进程中来说，对所有线程都是可见的，也就是说所有线程共享这个资源。这个对象中可能还有一些属性，是线程间的共享数据，那为了保证数据的安全性，对象中那些对属性进行操作的方法、代码，本来是所有线程共享的资源，现在要对其进行同步限制。

### Synchronized的使用方法

**`synchronized`可以用来修饰一个方法或者一个代码块甚至一个类，能够保证在同一时刻最多只有一个线程执行该段代码。**

我们来重写一下上面例子中的take()方法：

``` java
public synchronized void take(int x){     //模拟银行取款的方法，并加上synchronized标记
    sum-=x;
    System.out.println("sum="+sum);       //余额减去取款数后输出余额
}
```

运行结果：

    sum=1900
    sum=1800
    sum=1700
    sum=1600
    sum=1500
    sum=1400
    sum=1300
    sum=1200
    sum=1100
    sum=1000

程序运行结果符合预期，可以说线程是安全的。这是因为我们用`synchronized`关键字修饰了take()方法，对其进行了同步限制，所以在一个线程，比如c1线程执行完take()方法之前，c2线程不能进入take()方法，执行代码对共享数据sum进行访问，从而避免了一个线程对共享数据的修改结果覆盖了另一线程对共享数据的修改结果。那么，共享的数据sum就应该要设置为`private`权限，不然的话，通过**对象名.属性名**就能访问到sum，这样对共享数据的保护就没有意义了。

`synchronized`也可以这样用：

``` java
public void take(int x){                  //模拟银行取款的方法
    synchronized(this){                   //指定要锁定的对象，通常直接写this，后面接访问共享数据、需要互斥操作的代码块
        sum-=x;
        System.out.println("sum="+sum);   //余额减去取款数后输出余额
    }
}
```

运行结果跟上面一样。

我们可以这样理解：c1线程获得了bank对象的锁之后，由于bank对象是静态的，属于Mbank类，所以是唯一的，那对象锁只有一个，c2线程只能等待锁被释放掉，才能去访问bank对象中被`synchronized`标记或者说保护起来的共享资源。

那我们把bank对象的`static`去掉呢？

``` java
private Mbank bank=new Mbank();
```

运行结果：

    sum=1900
    sum=1900
    sum=1800
    sum=1800
    sum=1700
    sum=1700
    sum=1600
    sum=1600
    sum=1500
    sum=1500

程序运行结果又不符合预期了，这是因为去掉了`static`之后，每创建一个线程对象，在线程对象内部又会各自实例化自己的bank对象，这样每个线程都拿到了bank对象的锁，但不是同一个bank对象的锁，而是自己的bank对象的锁。

**所以，要理解“互斥锁”的含义，它锁的是一个实例化对象，一个对象只有一个自己的互斥锁，利用这种对锁的争夺，就可以实现不同线程间的互斥效果了。**

除了对象锁之外，还有另一种类型的锁，即`类锁`，一般用于两个地方：

1. `synchronized`用在类声明中，则表示该类中的所有方法都是synchronized的。

2. 一般用于标记`static`方法，即类方法。static方法要么整个是synchronized的，要么整个不是synchronized的，不能是其中一段代码被标记为synchronized。

### Synchronized的进一步说明

1. 被`synchronized`标记的代码块或方法，在各个线程中就不是并发执行了，而是串行执行了，因为互斥锁机制。

2. `synchronized`代码块中代码数量越少越好，包含的操作越少越好，因为没有锁的线程就不能执行这些操作，只能等待，白白浪费了CPU资源，这样就会丧失了多线程并发执行的性能优势。

3. **同一个对象中的`非synchronized`方法或代码块，都还是可以自由调用。即使一个线程获得了这个对象的锁，其它线程还是能自由调用该对象的所有非synchronized方法和代码块。**

4. **任何一个时刻，一个对象的互斥锁只能被一个线程拥有。只有当线程执行完它所调用的synchronized方法和代码之后，锁才会被释放掉。**

5. synchronize代码中要访问的共享数据，应该要设为`private`，原因上面讲过。

## 总结

这篇博客简单讲解了一下并发编程的基本原理和实现方法，以及synchronized。事实上，Java不止一种同步控制的方法，synchronized只是一种在早期并发编程中解决同步问题的一种方法，但也需要我们去好好理解synchronized，才能进一步学习好并发编程。