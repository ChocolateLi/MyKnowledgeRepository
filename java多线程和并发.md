# Java多线程和并发面试题

## 1.有多少种实现线程的方法？

1. 从不同的角度来看有不同的答案。比如说有继承Thread类、实现Runnable接口、实现Callable接口、线程池来创建

2. 但是典型的创建方法就只有继承Thread类和实现Runnable接口

3. 点开Thread类源码发现，其实Thread类继承了Runnable接口，看Thread类的run()方法，本质都是一样的

   ```java
   @Override
   public void run(){
       if(target!=null){
           target.run();
       }    
   }
   ```

   也就是说，“继承Thread然后重写run()方法”和“实现Runnable接口并传入Thread”在实现多线程的方式上并没有本质区别，最后都是调用start来创建线程

4. 其它实现线程的方法，线程池、定时器，细看源码，最后还是逃脱不了本质，最终还是实现Runnable接口或者继承Thread类

5. 结论：其实我们只有一种实现线程的方法，那就是通过Thread类新建线程。但是实现run()方法有两种，第一种是重写run()，第二种是实现Runnable接口的方法，再把Runnable接口的实例传给Thread类。除此之外，其实实现线程的方式，都最终逃脱不了这个范围

## 2.实现Runnable接口和继承Thread类哪个更好？

实现Runnable接口会更好。从三方面去说

1. 代码架构角度：实现Runnable接口可以更好地实现解耦，通过Runnable接口重写run()方法是具体执行的任务，可以让单独一个任务类实现Runnable接口，把实例传给Thread类。并且任务类不负责创建线程的工作，可以把任务类传给不同的Thread类，从而实现解耦。
2. 从资源消耗角度：继承Thread类，每次新建任务时，都要单独新建一个线程，这样开销是很大的。从线程创建到结束销毁。然而工作的内容只是run()函数里面的内容，这种开销是很大的。所以使用Runnable接口和线程池可以很好地减小损耗
3. 从扩展性来看：java是不支持多继承的，继承了Thread类就不能继承其他类了，限制了扩展性。


## 3.一个线程两次调用start()方法会出现什么情况？为什么？

答：会抛出异常。start的时候会检查线程的状态，只有NEW状态下的线程才可以继续，否则会抛出异常。

启动流程：

1. 先检查线程状态，只有NEW状态下线程才可以继续，否则抛出异常
2. 然后加入线程组
3. 最后调用start0()方法启动线程

**既然Thread.start()只能调用一次，那么线程池是如何实现复用的？**

线程复用的核心关键在于对Thread类进行了包装，它没有重复调用start()方法，而是自己有一个Runnable.run()方法。它在不断地跑，然后不断地检查是否有新的Runnable对象加入，如果有就调用一下run()方法。简单理解就是一个大run()，里面包含着小run()。同一个Thread可以执行不同的Runnable，主要原因是线程和Runnable通过BlockingQueue解耦，线程可以从BlockingQueue中不断获得新的任务。

## 4.既然start()方法会调用run()，为什么不直接调用run()方法？

start()方法才是启动线程的方法，如果调用run()方法，其实跟调用普通方法没有任何区别，跟线程生命周期一点关系都没有

## 5.如何正确停止线程？（难点）

1. 使用interupt()方法请求而不是强制，好处是安全
2. 这个需要三方的配合：请求方、被停止方、子方法被调用方
   - 请求方：请求中断
   - 被停止方：在循环中或者实时检查中断信号，在会抛出InterupException的地方处理异常
   - 子方法被调用方：不应该屏蔽中断，正确处理中断的措施有：1、传递中断。在方法中抛出异常  2、恢复中断。如果try\catch中捕获到异常，应该再次设置中断
3. stop/suspend()方法是被弃用的方法，volatile的boolean方法无法处理长时间阻塞的情况


## 6.如何处理不可中断的线程？

很遗憾，并没有通用的方法。如果不支持响应中断，那么只能用特定的方法唤起，没有万能的方法。（PS：线程阻塞是由于调用了wait()、sleep()或join()方法，设置中断，抛出InteruptException异常来唤醒阻塞的该线程）

## 7.线程有几种状态？生命周期是怎样的？

![线程的6种状态](https://github.com/ChocolateLi/MyKnowledge/blob/main/picture/%E7%BA%BF%E7%A8%8B%E7%9A%846%E4%B8%AA%E7%8A%B6%E6%80%81.png)

## 8.使用两个线程交替打印0~100奇偶数

1.使用synchronized方法

```java
package com.Thread.threadcoreknowledge.threadobjectcommonmethod;

/**
 * 描述：使用synchronized方法来回交替打印奇偶数
 */
public class WaitNotifyPrintOddEvenSyn {

    private static int count = 0;
    private static Object lock = new Object();

    public static void main(String[] args) {
        Thread even_thread = new Thread(new Runnable() {
            @Override
            public void run() {
                while (count<=100){
                    synchronized (lock){
                        if((count & 1)==0){
                            System.out.println(Thread.currentThread().getName()+":"+count);
                            count++;
                        }
                    }
                }
            }
        },"偶数");
        Thread odd_thread = new Thread(new Runnable() {
            @Override
            public void run() {
                while (count<100){
                    synchronized (lock){
                        if((count & 1)==1){
                            System.out.println(Thread.currentThread().getName()+":"+count);
                            count++;
                        }
                    }
                }
            }
        },"奇数");

        even_thread.start();
        odd_thread.start();
    }

}

```

2.使用wait()和notify()方法

```java
package com.Thread.threadcoreknowledge.threadobjectcommonmethod;

/**
 * 描述：使用wait和notify来回交替打印奇偶数
 */
public class WaitNotifyPrintOddEveWait {

    private static int count = 0;
    private static Object lock = new Object();

    static class Odd_Even_Thread implements Runnable{

        @Override
        public void run() {
            synchronized (lock){
                while (count<=100){
                    System.out.println(Thread.currentThread().getName()+":"+count++);
                    lock.notify();
                    //如果任务还没结束，就让出当前这把锁
                    if(count<=100){
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
    }

    public static void main(String[] args) {
        Odd_Even_Thread odd_even_thread = new Odd_Even_Thread();
        Thread t1 = new Thread(odd_even_thread,"偶数");
        Thread t2 = new Thread(odd_even_thread,"奇数");
        t1.start();
        t2.start();

    }

}

```

## 9.使用wait实现生产者消费者模式

```java
package com.Thread.threadcoreknowledge.threadobjectcommonmethod;

import java.util.Date;
import java.util.LinkedList;

/**
 * 描述：生产者消费者模式，不能使用阻塞队列
 */
public class ProducerComsumerMode {

    public static void main(String[] args) {
        EvenStorage storage = new EvenStorage();
        Producer producer = new Producer(storage);
        Comsumer comsumer = new Comsumer(storage);
        Thread produce_thread = new Thread(producer);
        Thread comsumer_thread = new Thread(comsumer);
        produce_thread.start();
        comsumer_thread.start();
    }
}

class EvenStorage{
    private int maxSize;
    private LinkedList<Date> storage;

    public EvenStorage() {
        maxSize = 10;
        storage = new LinkedList<>();
    }

    public synchronized void put(){
        while (storage.size()==maxSize){
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        storage.add(new Date());
        System.out.println("仓库里有"+storage.size()+"件产品");
        notify();
    }

    public synchronized void take(){
        while (storage.size()==0){
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("拿到了"+storage.poll()+"，仓库现在还剩"+storage.size()+"件产品");
        notify();
    }
}

class Producer implements Runnable{

    public EvenStorage storage;

    public Producer(EvenStorage storage) {
        this.storage = storage;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            storage.put();
        }
    }
}

class Comsumer implements Runnable{

    private EvenStorage storage;

    public Comsumer(EvenStorage storage) {
        this.storage = storage;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {

            storage.take();
        }
    }
}

```

生产者消费者模式是通过一个容器来解决生产者和消费者强耦合问题。生产者和消费者之间不直接通信，而是通过阻塞队列进行通信。生产者生产好数据之后放进阻塞队列里，消费者从阻塞队列里取数据消费。阻塞队列相当于一个缓冲区，平衡了生产者和消费者的处理能力。

## 10.为什么wait必须在同步代码块使用？

主要是为让通信变得可靠。通过同步代码可以保证notify()方法永远不会在wait()方法之前调用



## 11.为什么通信的方法wait()、notify()、notifyAll()定义在Object类中？而sleep()方法定义在Thread类中？（难点）

这个问题比较难回答，那我就说说我的理解吧！wait()、notify()、notifyAll()是属于锁级别的操作，而锁呢是属于某一个对象的而不是属于线程。所以把它们定义在Object类中，这样的话，Java中的每一个类都有线程通信的基本方法。



## 12.wait方法属于Object对象的，那么调用Thread.wait()会怎么样？

当Thread类当成普通的类来使用，和Object类是没什么区别。但是，使用Thread.wait方法之后，线程退出的时候会自动调用notify()方法，这就会导致自己设计的唤醒流程受到干扰，所以是不推荐用Thread.wait方法的。



## 13.如何选择使用notify()还是notifyAll()？

Object.notify()可能会导致信号丢失的正确性问题，Object.notifyAll()虽然效率不高（把不需要唤醒的线程给唤醒了），但它能保证信号不丢失的问题。所以实现通知的比较保守的方式是使用notifyAll()以保证正确性。

当然如果能证明以下两个条件，是可以用notify()方法来替代notifyAll()方法。1、一次通知至多唤醒一个线程 2、相应的对象上等待的线程是同质等待线程。所谓的同质等待线程是使用同一个保护条件。比如说同一个Runable接口创建的多个实例，或者同一个继承Thread类的子类new出来的多个实例


## 14.wait和sleep的异同？

相同点：

1. wait和sleep都会使线程阻塞，对应的线程状态是Waiting或者是Time_Waiting
2. 他们都可以响应中断Thread.interrupt()

不同点：

1. wait定义在Object类中，sleep定义在Thread类中
2. wait方法必须在同步代码块中执行，而sleep不需要
3. 在同步代码块中wait会主动释放锁，而sleep不会
4. sleep短暂休眠后会自动退出阻塞，而wait方法如果没设置时间的话，需要被其他线程唤醒或者中断来退出阻塞

## 15.join期间，线程处于什么状态？

join期间，线程处于Waitng状态，为什么不是Time_Waiting，因为join期间无法预料实际等待时间。


## 16.yield和sleep的区别？

yield主动释放自己的cpu时间片，此时的线程仍然处于就绪状态，随时可以被cpu调度。sleep期间，线程处于阻塞状态，线程调度器不会去调用该线程。


## 17.守护线程和普通线程的区别

没有什么区别，唯一不同的就是 JVM 的离开。如果用户线程全部退出运行，而还有守护线程还在运行，JVM还是会退出。因为没有了“被守护者”，所以也就没有被运行的必要了。


## 18.是否需要给线程设置为守护线程？

我们不应该把自己的线程设置为守护线程，因为设置守护线程是很危险的。比如我们访问数据库时，这时候所有的用户线程都退出了运行，那我们的线程也就中断。


## 19.为什么程序设计不应该依赖于线程优先级？

优先级是由线程调度器来决定的，依赖于操作系统。每个操作系统的优先级别不一样，优先级高的不一定比优先级低的先运行，也有可能导致线程饥饿的情况。

## 20.Java异常体系

![java异常体系](https://github.com/ChocolateLi/MyKnowledge/blob/main/picture/java%E5%BC%82%E5%B8%B8%E4%BD%93%E7%B3%BB.png)

所有异常都是继承Throwable，其下分为两个部分Error和Exception。Error级别的错误不是我们程序员能处理。Exception下面又分为RuntimeException和IOException等异常，其中RuntimeException的错误肯定是我们自己程序出现问题，需要及时去处理，比如数组越界异常和空指针异常。

## 21.如何全局处理异常？

我们可以自己实现一个类继承UncaughtExceptionHandler，在UncaughtException()方法中实现我们的代码逻辑，通常是把错误写进日志等。然后把我们的类注册到jvm中，抛出异常时调用我们的方法。


## 22.为什么异常要全局处理？不处理行不行？

不处理肯定是不行的。我们不应该把异常的信息抛给前端，这样会把重要的信息泄露，不安全。我们返回给前端一句简单的话就行，不应该把异常堆栈信息告诉它，不然会被黑客不法分子利用。


## 23.run()方法是否会抛出异常？如果抛出异常，线程状态会怎样？

run()方法不能抛出异常，如果是运行时异常，默认会打印异常堆栈信息，线程会停止运行，线程状态处于Terminated状态。不影响主线程的正常运行

## 24.讲讲什么是java内存模型

java内存模型是一组规范，它可以方便开发者利用这些规范来开发多线程程序。它最重要的三个部分就是重排序、可见性、原子性。

重排序：就是代码实际执行的顺序和代码在java中文件中的顺序不一致，也就是代码指令并不是严格按照代码语句顺序执行的，他们的顺序会改变，这就是重排序。

可见性：线程A内部对变量的执行操作在线程B内是可见的。因为所有共享变量存在于主内存中，每个线程有自己的本地内存，而且线程读写共享数据也是通过本地内存交换的，所以会导致可见性问题。

原子性：一组操作要么全部执行成功，要么全部失败。

## 25.什么是happens-before

happens-before规则是用来解决可见性问题，在时间上，动作A发生在动作B之前，保证B能看见A，这就是happens-before。

## 26.讲讲 volatile 关键字



