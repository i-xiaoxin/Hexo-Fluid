---
title: Java线程
date: 2022-10-11 20:09:02
tags: 
- Java
- 线程
index_img: /img/article2.webp
banner_img: /img/post_banner.webp
categories:
- Java
- 线程
comment: waline
---

## 一、进程与线程的概念

### 1.进程

进程的概念实在操作系统上体现的(电脑系统,手机),进程就是应用程序,例如:音乐,QQ,微信,王者荣耀,只要是应用程序并且打开了就是一个进程；执行中的程序就是`进程`,此`进程`是一个动态的概念-->进程中会有执行内容(运行状态,会有交互),并且赋予一定的独立功能；多进程就是在电脑的运行状态下可以打开多个应用程序开启多个进程,并且进程之间不会有相互干扰,并且可以同时运行,进程的运行要看的是CPU的核心数-->一个核心代表你可以开启一个进程(核心处理时什么样是最佳状态)-->CPU切换功能(当前在显示时主要显示的是当前打开的窗口)-->所以所有的进程都会抢占CPU的执行权-->所以软件程序开的越多越消耗CPU的资源

### 2.线程

线程的概念实在应用程序中提现的,并且应用程序(进程)想要执行就必须有一个线程(一般这个线程叫做主线程),线程的体现在应用程序运行

## 二、并发和并行的概念

### 1.并发

并发基本含义 在操作系统中，并发是指一个时间段中有几个程序都处于已启动运行到运行完毕之间，且这几个程序都是在同一个处理机上运行，但任一个时刻点上只有一个程序在处理机上运行

### 2.并行

并行是指“并排行走”或“同时实行或实施”。 在操作系统中是指，一组程序按独立异步的速度执行，无论从微观还是宏观，程序都是一起执行的。 对比地，并发是指:在同一个时间段内，两个或多个程序执行，有时间上的重叠(宏观上是同时,微观上仍是顺序执行)。

### 3.区别

- 并发:同时刻只能执行一条指令
- 并行:同时刻可以执行多条指令

## 三、Java中线程创建的几种方式

### 1.继承`Thread`类

> 步骤:
>
> - 创建一个类,继承Thread类
> - 实现Thread类中run方法-->run方法是多线程的执行体(线程执行的逻辑代码)
> - 创建测试类,并创继承了Thread类的类对象
> - 调用start方法启动线程


创建自定义类继承Thread类
```java
public class MyThread extends Thread {

    private String name;

    public MyThread() {
    }

    public MyThread(String name) {
        super(name);
        this.name = name;
    }

    /**
     * 如果想要实现多线程则必须重写run方法
     */
    @Override
    public void run() {
        // 注释父类的调用,自己实现
        // super.run();
        // 写一个循环代表多线程的执行逻辑
        for (int i = 0; i < 10; i++) {
            System.out.println(name + "线程执行:" + i);
        }
    }
}
```

创建测试方法并创建线程启动

```java
public class ThreadDemo {
    public static void main(String[] args) {
        // 创建多线程对象
        MyThread myThread1 = new MyThread("线程1");
        // 启动线程
        // 错误示范
        // myThread1.run();
        // 正确启动方式
        myThread1.start();
        // 创建多线程对象
        MyThread myThread2 = new MyThread("线程2");
        // 启动线程
        // 错误示范
        // myThread2.run();
        // 正确启动方式
        myThread2.start();
        System.out.println("程序结束...");
    }
}
```

### 2.实现Runnable接口重写run方法

> 步骤：
>
> - 创建线程类,并实现Runnable
> - 重写run方法并编写逻辑代码
> - 创建测试类并创建实现Runnable接口的类
> - 创建Thread,并将线程类做参数传入
> - 启动线程测试

线程类实现Runnable接口

```java
public class MyThread implements Runnable {

    /**
     * 多线程执行的逻辑的方法
     */
    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println("多线程执行:" + i);
        }
    }
}
```

线程测试类

```java
public class ThreadDemo {
    public static void main(String[] args) {
        // 创建线程类
        MyThread myThread = new MyThread();
        // 创建线程对象
        Thread thread1 = new Thread(myThread);
        // 创建线程对象
        Thread thread2 = new Thread(myThread);
        // 启动线程
        thread1.start();
        thread2.start();


        // 使用内部类创建多线程
        Thread thread3 = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    System.out.println("匿名内部类多线程执行:" + i);
                }
            }
        });
        // 启动线程
        thread3.start();
        System.out.println("程序结束...");
    }
}
```

### 3.实现Callable接口创建线程

> 步骤：
>
> - 创建线程类实现Callable接口,并实现call方法
> - 通过FutureTask类创建线程-->传入Callable的实现类
> - 创建Thread线程对象,并将FutureTask对象传入当做参数
> - 启动线程
> - 接收线程返回值

与前两不同,这里没有run方法,而是使用call方法去实现线程的逻辑代码,并且可以拥有`返回值`,但是接收返回值时会造成线程的阻塞(暂停等待接收数据),同样不能通过同一个FutureTask对象创建两个线程创建方式

创建线程类实现Callable接口

```java
/**
 * 实现Callable接口并指定泛型类型-->泛型类型是Value,并且是多线程的返回值
 */
public class MyThread implements Callable<String> {
    /**
     * call方法与之前的两种方式中的run方法相同,都是线程的执行体
     * 不同点在于call方法有返回值
     *
     * @return 返回值
     * @throws Exception 线程的异常
     */
    @Override
    public String call() throws Exception {
        for (int i = 0; i < 10; i++) {
            System.out.println("线程执行:" + i);
        }
        return "线程执行完毕";
    }
}
```

创建测试类启动线程

```java
public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        // 创建线程
        // 1. 创建实现Callable接口的类
        MyThread myThread = new MyThread();
        // 2. 使用FutureTask创建此线程,泛型类型与Callable实现时的相同
        // 参数为Callable实现类
        FutureTask<String> futureTask1 = new FutureTask<>(myThread);
        FutureTask<String> futureTask2 = new FutureTask<>(myThread);
        // 3. 创建线程类
        Thread thread1 = new Thread(futureTask1);
        // 4. 启动线程
        thread1.start();
        // 3. 创建线程类
        Thread thread2 = new Thread(futureTask2);
        // 4. 启动线程
        thread2.start();
        // 5. 接收线程返回的参数使用public V get() throws InterruptedException, ExecutionException方法
        // get方法会造成程序阻塞(会停止),等待接收线程返回的数据
        String result1 = futureTask1.get();
        System.out.println(result1);
        String result2 = futureTask2.get();
        System.out.println(result2);
        // 最后输出程序结束
        System.out.println("程序结束...");
    }
}
```

### 4.三种方式优缺

> 继承Thread类

优点:继承后直接可以启动线程

缺点:Java是单继承的语言,一旦继承了则无法继承其他的类的,造成我们的可扩展性非常低,并且没有返回值

> 实现Runnable接口

优点:解决了单继承的扩展性低缺点

缺点:没有返回值,需要单独创建Thread线程类启动

> 实现Callable接口创建FutureTask对象

优点:带有返回值并且也是通过接口实现方式创建

缺点:使用麻烦,并且接收返回值会造成程序阻塞(暂停),并且一个FutureTask对象只能启动一个线程

## 四、Java中线程的生命周期以及状态控制

### 1.线程的生命周期概述

<img src="https://raw.githubusercontent.com/i-xiaoxin/image/master/image-20221010212008318.png" style="zoom: 67%;" />

- 新建状态(New)

    使用new关键字创建了Thread线程类之后并且在调用start()方法之前就是新建状态,一旦调用start()方法则进入就绪状态

- 就绪状态(Runnable)

    调用start()方法后并且具备所有的可执行状态(没有错误的情况下)就是就绪状态-->一旦进入就绪状态后则能够去争夺CPU的执行权,仅仅只是争夺并没有执行权,所有的线程都需要去争夺线程执行权之后才能进入到运行状态

- 运行状态(Running)

    当处于就绪状态的线程争夺到CPU的执行权后即可进入并且执行run方法的逻辑(执行run方法就是执行状态)这时才是运行状态
    如果在运行状态时丢失CPU的执行权则进入就绪状态,如果调用sleep/join/等待同步锁/执行IO操作/wait方法时是进入阻塞状态
    如果调用yield方法则也会重新进入就绪状态

- 阻塞状态(Blocked)

    当运行状态的run方法中执行sleep/join/等待同步锁/执行IO操作/wait方法时则会出现阻塞状态(相当于执行途中暂停),这种状态时同样也是丢失CPU的执行权

    - 等待阻塞:运行中调用wait方法则是代表进入等待阻塞状态,只有等其他线程调用`notify`或`notifyAll`方法时才会被唤醒重新进入就绪状态
    - 同步等待:当获取执行等待锁时会进入同步等待-->当前某个对象有其他线程正在执行,只能等待其他线程执行完毕后才允许获取,获取同步锁后才会重新进入到就绪状态
    - 其他状态:当线程代用join/IO操作时,join线程让步代表当前这次进入阻塞状态让其他先行,方法调用完毕后则重新进入就绪状态;IO操作由于是针对文件的操作,会循环读取或者读写时会造成程序阻塞(在读写数据时有一个过程,这个过程就是造成程序阻塞原因)

- 消亡状态(Dead)

    线程run方法执行完毕后就是线程的消亡

### 2.线程的休眠

- `public static native void sleep(long millis) throws InterruptedException`:线程休眠millis是代表毫秒值
- `public static void sleep(long millis, int nanos) throws InterruptedException`:线程休眠millis是毫秒值,nanos纳秒值

```java
// 创建线程
public class MyThread extends Thread {

    public MyThread(String name) {
        super(name);
    }

    @Override
    public void run() {
        // 线程体
        for (int i = 0; i < 10; i++) {
            // 线程run方法不允许声明抛出异常必须捕获
            try {
                // 让线程休眠一小会儿
                if (i == 5) {
                    Thread.sleep(1000);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 输出线程执行的流程
            System.out.println(this.getName() + "-线程执行:" + i);
        }
    }
}
```

```java
// 线程休眠测试类
public class A线程休眠 {
    public static void main(String[] args) throws InterruptedException {
        // 创建自定义线程
        MyThread myThread1 = new MyThread("线程休眠1");
        MyThread myThread2 = new MyThread("线程休眠2");
        // 启动线程
        myThread1.start();
        myThread2.start();


        System.out.println("主线程休眠开始...");
        // 线程休眠
        Thread.sleep(5000);
        System.out.println("主线程休眠结束...");
    }
}
```

### 3.线程的优先级

每个线程执行时都有一个优先级,设置优先级可以让CPU优先选择,但是CPU选不选择不光是看优先级的-->虽然可以设置但是最终的执行结果不一定是优先级最高的

- `public final void setPriority(int newPriority)`:设置线程优先级别优先级数值为1-10,默认是5

- Thread类静态常量;以下三个静态常量是Thread类提供设置线程优先级时使用
    -  最低优先级 		  public final static int MIN_PRIORITY = 1;
    -  中等优先级(默认)  public final static int NORM_PRIORITY = 5;
    - 最高优先级           public final static int MAX_PRIORITY = 10;

```java
public class MyThread implements Runnable {

    // 获取当前执行的线程对象-->静态方法
    // private Thread thread;
    private int priority;

    public MyThread() {

    }

    public MyThread(int priority) {
        // 获取当前线程对象
        // thread = Thread.currentThread();
        // 设置优先级
        // thread.setPriority(priority);
        // 设置线程优先级
        this.priority = priority;
    }

    /**
     * 线程执行方法
     */
    @Override
    public void run() {
        // 获取当前线程对象
        Thread thread = Thread.currentThread();
        // 设置线程优先级-->不切实际,原因是run方法不止直接调用,而是JVM虚拟机调用,抢占到CPU执行权时才执行run方法
        thread.setPriority(this.priority);
        // 线程执行
        for (int i = 0; i < 10; i++) {
            System.out.println(thread.getName() + "-线程执行:" + i);
        }
    }
}
```

```java
public class Demo {
    public static void main(String[] args) {
        // 创建线程
        MyThread myThread = new MyThread();
        // 创建线程对象用于执行线程
        Thread thread1 = new Thread(myThread, "线程1");
        Thread thread2 = new Thread(myThread, "线程2");
        Thread thread3 = new Thread(myThread, "线程3");
        // 设置线程优先级
        // 以下三个常量是Thread类提供的设置线程优先级时使用的
        // Thread.MIN_PRIORITY
        // Thread.NORM_PRIORITY
        // Thread.MAX_PRIORITY
        // 设置最低优先级
        thread1.setPriority(Thread.MIN_PRIORITY);
        // 设置中等优先级
        thread2.setPriority(Thread.NORM_PRIORITY);
        // 设置最高优先级
        thread3.setPriority(Thread.MAX_PRIORITY);
        // 启动线程测试
        thread1.start();
        thread2.start();
        thread3.start();
    }
}
```

### 4.线程让步

线程让步是当前正在执行的线程处于孔融让梨精神放弃此次执行的机会,让给其他线程执行,但是让归让不代表我不进行争夺,有可能会出现我让步了我有不小心争夺到了执行权;线程让步会让当前线程重新进入就绪状态

- `public static native void yield()`:线程让步方法是静态方法,调用的是底层

- 让步线程类

    ```java
    public class MyThread01 implements Runnable {
    
        @Override
        public void run() {
            // 获取当前执行的线程对象
            Thread thread = Thread.currentThread();
            for (int i = 0; i < 10; i++) {
                // 调用线程让步方法-->静态方法直接调用
                Thread.yield();
                System.out.println(thread.getName() + "-线程执行:" + i);
            }
        }
    }
    ```

- 不让步线程类

    ```java
    public class MyThread02 implements Runnable {
    
        @Override
        public void run() {
            // 获取当前执行的线程对象
            Thread thread = Thread.currentThread();
            for (int i = 0; i < 10; i++) {
                System.out.println(thread.getName() + "-线程执行:" + i);
            }
        }
    }
    ```

    测试代码

``` java
public class Demo {
    public static void main(String[] args) {
        // 创建线程对象
        // 创建线程1,是存在线程让步的
        MyThread01 myThread1 = new MyThread01();
        // 创建线程2,是没有线程让步的
        MyThread02 myThread2 = new MyThread02();
        // 创建线程对象
        Thread thread1 = new Thread(myThread1, "让步线程");
        Thread thread2 = new Thread(myThread2, "不让步线程");
        // 启动线程
        thread1.start();
        thread2.start();
        // main方法中调用线程让步main方法会让
        Thread.yield();
        System.out.println("main方法中线程让步");
    }
}
```

### 5.线程合并(线程加入插入)

当前有两个线程,线程A和线程B,比如执行线程时先让B执行,并且线程B调用join插入其中,这时线程A就必须等待线程B的执行结束;相当于线程A和线程B执行时,线程A充了钱,线程会在线程B直线执行,并且线程B会等待线程A的执行完毕

- `public final void join() throws InterruptedException`:线程合并(或者叫插入)
- `public final synchronized void join(long millis) throws InterruptedException`:线程合并(或者叫插入),插入多少毫秒然后离开
- `public final synchronized void join(long millis, int nanos) throws InterruptedException`:线程合并(或者叫插入),插入多少毫秒/纳秒然后离开

```java
public class MyThread extends Thread {

    public MyThread(String name) {
        super(name);
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(this.getName() + "-线程执行:" + i);
        }
    }
}
```

```java
public class Demo {
    public static void main(String[] args) {
        // 创建线程对象
        MyThread myThread01 = new MyThread("线程1");
        // 启动线程
        myThread01.start();
        // 插入线程
        try {
            // 调用插入合并线程方法
            myThread01.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 主线程执行for循环
        for (int i = 0; i < 10; i++) {
            System.out.println("主线程执行:" + i);
        }
    }
}
```

### 6.守护线程

守护线程存在意义是为了守护程序中的线程,并且守护线程不会主动死亡(只要执行了并且给了一个死循环则不会主动结束),只有当所有线程(非守护线程外的)全部消亡后才会跟随死亡
守护线程可以理解是后台线程-->在后台默默的工作的线程,无私奉献的
如果想让普通线程切换为守护线程只需要调用方法`setDaemon`并且必须是在调用`start`方法前调用

- `public final void setDaemon(boolean on)`:将当前线程设置为守护线程

守护线程类

```java
public class MyThread extends Thread {
    @Override
    public void run() {
        int index = 0;
        while (true) {
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("守护线程执行:" + (index++));
        }
    }
}
```

普通线程类

```java
public class MyThread01 extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程执行:" + i);
        }
    }
}
```

测试类

```java
public class Demo {
    public static void main(String[] args) throws InterruptedException {
        // 创建守护线程对象
        MyThread myThread = new MyThread();
        // 将当前线程设置为守护线程,setDaemon此方法值为true时是守护线程,为false时是非守护线程(普通线程)
        myThread.setDaemon(true);
        // 启动线程
        myThread.start();

        // 创建普通线程
        MyThread01 myThread01 = new MyThread01();
        // 启动普通线程
        myThread01.start();

        // 手动睡一下
        Thread.sleep(2000);


        // main方法执行循环
        for (int i = 0; i < 100; i++) {
            System.out.println("main方法执行:" + i);
        }
    }
}
```

## 五、线程同步synchronized

### 1.synchronized关键字注意事项

- synchronized只能修饰方法和代码块,不能修饰变量和类
- synchronized只有在多线程读取同一个共享数据时才需要使用,如果多线程不操作共享资源(同一个资源)则不需要使用synchronized关键字修饰
- 经过synchronized关键字修饰后不管是方法还是代码执行效率都会降低,并且还会出现死锁问题
- 同步方法的同步监视器是`this`对象,静态同步方法的同步监视器是`类名.class`类,同步代码块的同步监视器是自定义的(一般锁住的是共享资源)
- 同步方法和静态同步方法是将整个方法体锁住,甭管里面是否是所有的代码都需要锁住;同步代码块是只需要锁住共享资源即可能稍微的提高效率

### 2.synchronized关键字时是什么时候获取锁什么时候释放锁

- 同步方法和静态同步方法是在调用方法时大括号的起始位开始锁,大括号的结束位置是释放锁-->方法执行完毕才会释放同步锁
- 同步代码块是在同步代码块执行时开始锁,执行结束时释放同步锁
- 如果发生异常或者使用return/break关键字时也会提前结束
- 如果同步方法或同步代码块调用了wait()方法也会释放锁对象

### 3.synchronized关键字的可重入性

在使用一个同步方法时,是否可以调用另一个同步方法,就是俄罗斯套娃

```java
public class ThreadSynchronizedDemo {
    public static void main(String[] args) {
        // 调用静态同步方法
        func01();
    }

    public static synchronized void func01() {
        System.out.println("静态同步方法1");
        // 调用静态同步方法2
        func02();
    }

    public static synchronized void func02() {
        System.out.println("静态同步方法2");
        // 调用普通静态方法
        func03();
    }

    public static void func03() {
        System.out.println("普通静态方法3");
    }
}
```

## 六、线程同步的死锁

### 1.概述

线程死锁可能会出现的场景:假设有一个共享类,共享类中有两个属性,属性1和属性2,并且有两个线程,线程A和线程B,线程A操作共享类中的属性1和属性2,同样线程B也须要操作属性1和属性2,但是我们要求属性1和属性2他们两个都必须是安全的(数据不能出现问题),将他俩全部锁住线程1中锁住属性1并且在锁住属性1的同步代码块中锁属性2,相同的线程B再锁住属性2,在属性2的同步代码块中锁属性1,这样就可以实现死锁现象,原因是,当线程A拿到属性1的锁后会进入,在线程A拿到属性1的锁时线程B有可能已经拿到属性2的锁,拿到属性2的锁的时候同样会去拿属性1的锁,而这时属性1的锁在线程A手中,会等待,同样线程A想要拿属性2的锁时,属性2同样在线程B的手中,这样就造成了死锁问题

### 2. 代码演示死锁问题

共享资源类

```jav
/**
 * 共享资源类
 */
public class Source {
    // 两个共享属性
    public static Object object01 = new Object();
    public static Object object02 = new Object();
}
```

线程A

```jav
/**
 * 实现死锁的线程A
 */
public class ThreadA extends Thread {

    /**
     * 实现死锁
     */
    @Override
    public void run() {
        // 锁住属性1-->Object01
        synchronized (Source.object01) {
            System.out.println("ThreadA拿到object01的锁...");
            // 休眠10毫秒,目的是给线程B留下拿属性2锁的机会
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 取得属性2的锁
            synchronized (Source.object02) {
                System.out.println("ThreadA拿到object02的锁...");
            }
        }
    }
}
```

线程B

```java
/**
 * 实现死锁的线程B
 */
public class ThreadB extends Thread {

    /**
     * 实现死锁
     */
    @Override
    public void run() {
        // 锁住属性2-->Object02
        synchronized (Source.object02) {
            System.out.println("ThreadB拿到object02的锁...");
            // 休眠10毫秒,目的是给线程A留下拿属性1锁的机会
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 取得属性2的锁
            synchronized (Source.object01) {
                System.out.println("ThreadB拿到object01的锁...");
            }
        }
    }
}
```

测试类

```java
public class Demo {
    public static void main(String[] args) {
        // 测试线程死锁
        ThreadA threadA = new ThreadA();
        ThreadB threadB = new ThreadB();
        // 启动线程
        threadA.start();
        threadB.start();
    }
}
```

### 七、线程通讯（wait/notify/notifyAll）

### 1.生产者与消费者模式

生产者-->代表是商户

```java
/**
 * 生产者类,代表商户
 */
public class Producer {
    private String mealName;

    public String getMealName() {
        return mealName;
    }

    public void setMealName(String mealName) {
        this.mealName = mealName;
    }

    @Override
    public String toString() {
        return "Producer{" +
                "mealName='" + mealName + '\'' +
                '}';
    }
}
```

缓冲区-->外卖桌子

```java
/**
 * 缓冲区,放外卖的桌子-->给商户使用
 */
public class Buffer {

    // 商户对象-->放外卖
    private Producer producer;

    // 标记,用于商户做好外卖后证明自己是不是做了外卖
    // 也用于仓库中没有外卖的提示
    private boolean flag;// true:有外卖;false:没有外卖;

    /**
     * 通过构造方法创建生产者商户对象
     *
     * @param producer 生产者商户对象
     */
    public Buffer(Producer producer) {
        this.producer = producer;
    }

    /**
     * 生产外卖的方法
     *
     * @param mealName 做好的外卖是什么
     */
    public synchronized void producer(String mealName) {
        // 1. 验证当前外卖桌是否有外卖商品
        if (flag) {// true:有外卖;false:没有外卖;
            // 如果有外卖则不能生产外卖
            try {
                // wait方法代表等待,会让线程进入阻塞状态
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        // 2. 如果没有外卖则生产外卖
        this.producer.setMealName(mealName);
        // 输出一句话,为了执行时知道做好了
        System.out.println("---------生产者,商户做好了:" + producer.getMealName());
        // 3. 将flag状态改为true,代表外卖做好了
        flag = true;// true:有外卖;false:没有外卖;
        // 4. 唤醒外卖小哥可以取外卖了-->唤醒消费者
        this.notify();
    }

    // 消费外卖方法
    public synchronized void consumer() {
        // 1. 验证桌子上是否有外卖
        if (!flag) {// true:有外卖;false:没有外卖;
            try {
                // 如果没有外卖则需要等待
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        // 2. 如果有外卖则取走
        // 输出一句话代表取走了外卖
        System.out.println("消费者,外卖小哥取走了:" + this.producer.getMealName());
        // 3. 更改桌子状态
        flag = false;// true:有外卖;false:没有外卖;
        // 4. 唤醒生产者让他继续做外卖
        this.notify();
    }
}
```

生产者线程类-->生产商品

```java
/**
 * 生产者线程(商户)
 */
public class ProducerThread implements Runnable {

    // 缓冲区对象(做好外卖后放到桌子上)
    private Buffer buffer;

    // 通过构造方法创建缓冲区对象
    public ProducerThread(Buffer buffer) {
        this.buffer = buffer;
    }

    /**
     * run方法代表生产外卖的方法
     */
    @Override
    public void run() {
        // 死循环生产外卖
        for (int i = 0; ; i++) {
            if (i % 2 == 0) {
                // 调用缓冲区制作外卖
                buffer.producer("酱焖肘子");
                continue;
            }
            if (i % 3 == 0) {
                // 调用缓冲区制作外卖
                buffer.producer("铁锅炖大ne");
                continue;
            }
            if (i % 7 == 0) {
                // 调用缓冲区制作外卖
                buffer.producer("猪肉炖粉条");
                continue;
            }
        }
    }
}
```

消费者线程类-->外卖小哥取外卖

```java
/**
 * 消费者线程
 */
public class ConsumerThread implements Runnable {

    // 缓冲区对象(取走外卖)
    private Buffer buffer;

    // 通过构造方法创建缓冲区对象
    public ConsumerThread(Buffer buffer) {
        this.buffer = buffer;
    }

    /**
     * 取走外卖的方法
     */
    @Override
    public void run() {
        // 死循环
        while (true) {
            // 在缓冲区取走外卖
            buffer.consumer();
        }
    }
}
```

测试类

```java
public class Demo {
    public static void main(String[] args) {
        // 1. 创建商家-->生产者
        Producer producer = new Producer();
        // 2. 创建缓冲区(外卖桌子)
        Buffer buffer = new Buffer(producer);
        // 3. 创建线程
        ProducerThread producerThread = new ProducerThread(buffer);
        ConsumerThread consumerThread = new ConsumerThread(buffer);
        // 4. 启动线程
        new Thread(producerThread).start();
        new Thread(consumerThread).start();

    }
}
```

### 2.生产者与消费者存在问题

> 上面在使用生产者与消费者时是单线程,正常交替,如果说出现两条线程同时生产并且同时消费,那这时会不会出现问题????
> 一旦出现了多生产者和多消费者后就会造成会有一方取走|制作多次
> 以上问题解决方案,将原本if改为while循环判断是否存在商品或外卖

### 解决多取|多做问题

生产者-->代表是商户

```java
/**
 * 生产者类,代表商户
 */
public class Producer {
    private String mealName;

    public String getMealName() {
        return mealName;
    }

    public void setMealName(String mealName) {
        this.mealName = mealName;
    }

    @Override
    public String toString() {
        return "Producer{" +
                "mealName='" + mealName + '\'' +
                '}';
    }
}
```

缓冲区-->外卖桌子

```java
/**
 * 缓冲区,放外卖的桌子-->给商户使用
 */
public class Buffer {

    // 商户对象-->放外卖
    private Producer producer;

    // 标记,用于商户做好外卖后证明自己是不是做了外卖
    // 也用于仓库中没有外卖的提示
    private boolean flag;// true:有外卖;false:没有外卖;

    /**
     * 通过构造方法创建生产者商户对象
     *
     * @param producer 生产者商户对象
     */
    public Buffer(Producer producer) {
        this.producer = producer;
    }

    /**
     * 生产外卖的方法
     *
     * @param mealName 做好的外卖是什么
     */
    public synchronized void producer(String mealName) {
        // 1. 验证当前外卖桌是否有外卖商品
        // 将原本的if判断一次改为while循环的多次判断验证
        while (flag) {// true:有外卖;false:没有外卖;
            // 如果有外卖则不能生产外卖
            try {
                // wait方法代表等待,会让线程进入阻塞状态
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        // 2. 如果没有外卖则生产外卖
        this.producer.setMealName(mealName);
        // 输出一句话,为了执行时知道做好了
        System.out.println("---------生产者,商户做好了:" + producer.getMealName());
        // 3. 将flag状态改为true,代表外卖做好了
        flag = true;// true:有外卖;false:没有外卖;
        // 4. 唤醒外卖小哥可以取外卖了-->唤醒消费者
        this.notify();
    }

    // 消费外卖方法
    public synchronized void consumer() {
        // 1. 验证桌子上是否有外卖
        // 将原本的if判断一次改为while循环的多次判断验证
        while (!flag) {// true:有外卖;false:没有外卖;
            try {
                // 如果没有外卖则需要等待
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        // 2. 如果有外卖则取走
        // 输出一句话代表取走了外卖
        System.out.println("消费者,外卖小哥取走了:" + this.producer.getMealName());
        // 3. 更改桌子状态
        flag = false;// true:有外卖;false:没有外卖;
        // 4. 唤醒生产者让他继续做外卖
        this.notify();
    }
}
```

生产者线程类-->生产商品

```java
/**
 * 生产者线程(商户)
 */
public class ProducerThread implements Runnable {

    // 缓冲区对象(做好外卖后放到桌子上)
    private Buffer buffer;

    // 通过构造方法创建缓冲区对象
    public ProducerThread(Buffer buffer) {
        this.buffer = buffer;
    }

    /**
     * run方法代表生产外卖的方法
     */
    @Override
    public void run() {
        // 死循环生产外卖
        for (int i = 0; ; i++) {
            if (i % 2 == 0) {
                // 调用缓冲区制作外卖
                buffer.producer("酱焖肘子");
                continue;
            }
            if (i % 3 == 0) {
                // 调用缓冲区制作外卖
                buffer.producer("铁锅炖大ne");
                continue;
            }
            if (i % 7 == 0) {
                // 调用缓冲区制作外卖
                buffer.producer("猪肉炖粉条");
                continue;
            }
        }
    }
}
```

消费者线程类-->外卖小哥取外卖

```java
/**
 * 消费者线程
 */
public class ConsumerThread implements Runnable {

    // 缓冲区对象(取走外卖)
    private Buffer buffer;

    // 通过构造方法创建缓冲区对象
    public ConsumerThread(Buffer buffer) {
        this.buffer = buffer;
    }

    /**
     * 取走外卖的方法
     */
    @Override
    public void run() {
        // 死循环
        while (true) {
            // 在缓冲区取走外卖
            buffer.consumer();
        }
    }
}
```

测试类

```java
public class Demo {
    public static void main(String[] args) {
        // 1. 创建商家-->生产者
        Producer producer = new Producer();
        // 2. 创建缓冲区(外卖桌子)
        Buffer buffer = new Buffer(producer);
        // 3. 创建线程
        ProducerThread producerThread = new ProducerThread(buffer);
        ConsumerThread consumerThread = new ConsumerThread(buffer);
        // 4. 启动线程
        new Thread(producerThread).start();
        new Thread(consumerThread).start();

    }
}
```

以上代码解决了多做|多取的问题了,但是使用的是notify方法,在唤醒时有可能唤醒不是对方而是己方,一旦唤醒是己方则会出现死锁(都在wait等待唤醒)

### 解决多做|多取的问题并且解决死等(死锁)问题

生产者-->代表是商户

```java
/**
 * 生产者类,代表商户
 */
public class Producer {
    private String mealName;

    public String getMealName() {
        return mealName;
    }

    public void setMealName(String mealName) {
        this.mealName = mealName;
    }

    @Override
    public String toString() {
        return "Producer{" +
                "mealName='" + mealName + '\'' +
                '}';
    }
}
```

缓冲区-->外卖桌子

```java
/**
 * 缓冲区,放外卖的桌子-->给商户使用
 */
public class Buffer {

    // 商户对象-->放外卖
    private Producer producer;

    // 标记,用于商户做好外卖后证明自己是不是做了外卖
    // 也用于仓库中没有外卖的提示
    private boolean flag;// true:有外卖;false:没有外卖;

    /**
     * 通过构造方法创建生产者商户对象
     *
     * @param producer 生产者商户对象
     */
    public Buffer(Producer producer) {
        this.producer = producer;
    }

    /**
     * 生产外卖的方法
     *
     * @param mealName 做好的外卖是什么
     */
    public synchronized void producer(String mealName) {
        // 1. 验证当前外卖桌是否有外卖商品
        // 将原本的if判断一次改为while循环的多次判断验证
        while (flag) {// true:有外卖;false:没有外卖;
            // 如果有外卖则不能生产外卖
            try {
                // wait方法代表等待,会让线程进入阻塞状态
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        // 2. 如果没有外卖则生产外卖
        this.producer.setMealName(mealName);
        // 输出一句话,为了执行时知道做好了
        System.out.println("---------生产者,商户做好了:" + producer.getMealName());
        // 3. 将flag状态改为true,代表外卖做好了
        flag = true;// true:有外卖;false:没有外卖;
        // 4. 唤醒外卖小哥可以取外卖了-->唤醒消费者
        // 由于每次唤醒一个会出现将同伴唤醒情况,所以直接将所有在等待线程全都唤醒即可,所以将notify改为notifyAll
        // this.notify();
        this.notifyAll();
    }

    // 消费外卖方法
    public synchronized void consumer() {
        // 1. 验证桌子上是否有外卖
        // 将原本的if判断一次改为while循环的多次判断验证
        while (!flag) {// true:有外卖;false:没有外卖;
            try {
                // 如果没有外卖则需要等待
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        // 2. 如果有外卖则取走
        // 输出一句话代表取走了外卖
        System.out.println("消费者,外卖小哥取走了:" + this.producer.getMealName());
        // 3. 更改桌子状态
        flag = false;// true:有外卖;false:没有外卖;
        // 4. 唤醒生产者让他继续做外卖
        // 由于每次唤醒一个会出现将同伴唤醒情况,所以直接将所有在等待线程全都唤醒即可,所以将notify改为notifyAll
        // this.notify();
        this.notifyAll();
    }
}
```

生产者线程类-->生产商品

```java
/**
 * 生产者线程(商户)
 */
public class ProducerThread implements Runnable {

    // 缓冲区对象(做好外卖后放到桌子上)
    private Buffer buffer;

    // 通过构造方法创建缓冲区对象
    public ProducerThread(Buffer buffer) {
        this.buffer = buffer;
    }

    /**
     * run方法代表生产外卖的方法
     */
    @Override
    public void run() {
        // 死循环生产外卖
        for (int i = 0; ; i++) {
            if (i % 2 == 0) {
                // 调用缓冲区制作外卖
                buffer.producer("酱焖肘子");
                continue;
            }
            if (i % 3 == 0) {
                // 调用缓冲区制作外卖
                buffer.producer("铁锅炖大ne");
                continue;
            }
            if (i % 7 == 0) {
                // 调用缓冲区制作外卖
                buffer.producer("猪肉炖粉条");
                continue;
            }
        }
    }
}
```

消费者线程类-->外卖小哥取外卖

```java
/**
 * 消费者线程
 */
public class ConsumerThread implements Runnable {

    // 缓冲区对象(取走外卖)
    private Buffer buffer;

    // 通过构造方法创建缓冲区对象
    public ConsumerThread(Buffer buffer) {
        this.buffer = buffer;
    }

    /**
     * 取走外卖的方法
     */
    @Override
    public void run() {
        // 死循环
        while (true) {
            // 在缓冲区取走外卖
            buffer.consumer();
        }
    }
}
```

测试类

```java
public class Demo {
    public static void main(String[] args) {
        // 1. 创建商家-->生产者
        Producer producer = new Producer();
        // 2. 创建缓冲区(外卖桌子)
        Buffer buffer = new Buffer(producer);
        // 3. 创建线程
        ProducerThread producerThread = new ProducerThread(buffer);
        ConsumerThread consumerThread = new ConsumerThread(buffer);
        // 4. 启动线程
        new Thread(producerThread).start();
        new Thread(consumerThread).start();

    }
}
```

虽然以上方法可以解决线程死锁问题,但是会出现资源浪费的情况,原因是唤醒时没必要将所有的线程全都唤醒,唤醒时只需要唤醒对方即可



<div>
    <script src="//cdn.jsdelivr.net/npm/@waline/client"></script>
<script src="//cdn.jsdelivr.net/npm/@waline/client"></script>  
<div id="waline"></div>
  <script>
    Waline({
      el: '#waline',
      serverURL: 'https://vercel-project-4d7haxk1c-i-xiaoxin.vercel.app',
    });
  </script>
