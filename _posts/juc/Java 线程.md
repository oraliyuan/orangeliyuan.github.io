[toc]





### 线程与进程



进程：

* 程序由指令和数据组成，但这些要运行，数据要读写，就必须将指令加载至 CPU，数据加载至内存。在指令运行过程中还需要用到磁盘、网络等设备。进程就是用来加载指令、管理内存、管理 IO 的 。
* 当一个程序被运行，从磁盘加载这个程序的代码至内存，这时就开启了一个进程。 
* 进程就可以视为程序的一个实例。大部分程序可以同时运行多个实例进程，也有的程序只能启动一个实例进程。
* 操作系统会以进程为单位，分配系统资源（CPU时间片、内存等资源），进程是资源分配的最小单位。



线程：

* 线程是进程中的实体，一个进程可以拥有多个线程，一个线程必须有一个父进程。 
* 一个线程就是一个指令流，将指令流中的一条条指令以一定的顺序交给 CPU 执行 。
* 线程，有时被称为轻量级进程(Lightweight Process，LWP），是操作系统调度（CPU调度）执行的最小单位。



进程与线程的区别：

* 进程基本上相互独立的，而线程存在于进程内，是进程的一个子集
* 进程拥有共享的资源，如内存空间等，供其内部的线程共享
* 进程间通信较为复杂

- - 同一台计算机的进程通信称为 IPC
  - 不同计算机之间的进程通信，需要通过网络，并遵守共同的协议，例如 HTTP

- 线程通信相对简单，因为它们共享进程内的内存，一个例子是多个线程可以访问同一个共享变量
- 线程更轻量，线程上下文切换成本一般上要比进程上下文切换低



进程间通信的方式：

* **管道（pipe）及有名管道（named pipe）**：管道可用于具有亲缘关系的父子进程间的通信，有名管道除了具有管道所具有的功能外，它还允许无亲缘关系进程间的通信。
* **信号（signal）**：信号是在软件层次上对中断机制的一种模拟，它是比较复杂的通信方式，用于通知进程有某事件发生，一个进程收到一个信号与处理器收到一个中断请求效果上可以说是一致的。
* **消息队列（message queue）**：消息队列是消息的链接表，它克服了上两种通信方式中信号量有限的缺点，具有写权限得进程可以按照一定得规则向消息队列中添加新信息；对消息队列有读权限得进程则可以从消息队列中读取信息。
* **共享内存（shared memory）**：可以说这是最有用的进程间通信方式。它使得多个进程可以访问同一块内存空间，不同进程可以及时看到对方进程中对共享内存中数据得更新。这种方式需要依靠某种同步操作，如互斥锁和信号量等。
* **信号量（semaphore）**：主要作为进程之间及同一种进程的不同线程之间得同步和互斥手段。
* **套接字（socket）**：这是一种更为一般得进程间通信机制，它可用于网络中不同机器之间的进程间通信，应用非常广泛。



线程的同步互斥：

* **线程同步**是指线程之间所具有的一种制约关系，一个线程的执行依赖另一个线程的消息，当它没有得到另一个线程的消息时应等待，直到消息到达时才被唤醒。
* **线程互斥**是指对于共享的进程系统资源，在各单个线程访问时的排它性。当有若干个线程都要使用某一共享资源时，任何时刻最多只允许一个线程去使用，其它要使用该资源的线程必须等待，直到占用资源者释放该资源。线程互斥可以看成是一种特殊的线程同步。



上下文切换：

上下文切换是指CPU(中央处理单元)从一个进程或线程到另一个进程或线程的切换。

上下文是CPU寄存器和程序计数器在任何时间点的内容。



上下文切换可以更详细地描述为内核(即操作系统的核心)对CPU上的进程(包括线程)执行以下活动:

1. 暂停一个进程的处理，并将该进程的CPU状态(即上下文)存储在内存中的某个地方
2. 从内存中获取下一个进程的上下文，并在CPU的寄存器中恢复它
3. 返回到程序计数器指示的位置(即返回到进程被中断的代码行)以恢复进程。



上下文切换只能在内核模式下发生。内核模式是CPU的特权模式，其中只有内核运行，并提供对所有内存位置和所有其他系统资源的访问。其他程序(包括应用程序)最初在用户模式下运行，但它们可以通过系统调用运行部分内核代码。



内核态：**Kernel Mode**

在内核模式下，执行代码可以完全且不受限制地访问底层硬件。它可以执行任何CPU指令和引用任何内存地址。内核模式通常为操作系统的最低级别、最受信任的功能保留。内核模式下的崩溃是灾难性的;他们会让整个电脑瘫痪。

用户态：**User Mode**
在用户模式下，执行代码不能直接访问硬件或引用内存。在用户模式下运行的代码必须委托给系统api来访问硬件或内存。由于这种隔离提供的保护，用户模式下的崩溃总是可恢复的。在您的计算机上运行的大多数代码将在用户模式下执行。

![image-20211229000129852](/Users/oraliyuan/Library/Application Support/typora-user-images/image-20211229000129852.png)



上下文切换是多任务操作系统的一个基本特性。在多任务操作系统中，多个进程似乎同时在一个CPU上执行，彼此之间互不干扰。这种并发的错觉是通过快速连续发生的上下文切换(每秒数十次或数百次)来实现的。这些上下文切换发生的原因是进程自愿放弃它们在CPU中的时间，或者是调度器在进程耗尽其CPU时间片时进行切换的结果。



上下文切换通常是计算密集型的。就CPU时间而言，上下文切换对系统来说是一个巨大的成本，实际上，它可能是操作系统上成本最高的操作。因此，操作系统设计中的一个主要焦点是尽可能地避免不必要的上下文切换。与其他操作系统(包括一些其他类unix系统)相比，Linux的众多优势之一是它的上下文切换和模式切换成本极低。



```sh
# 查看整个操作系统每1秒CPU上下文切换的统计
vmstat 1
```



操作系统层面线程的生命周期

初始状态、可运行状态、运行状态、休眠状态和终止状态。

![image-20211229000448089](/Users/oraliyuan/Library/Application Support/typora-user-images/image-20211229000448089.png)



1. 初始状态，指的是线程已经被创建，但是还不允许分配 CPU 执行。这个状态属于编程语言特有的，不过这里所谓的被创建，仅仅是在编程语言层面被创建，而在操作系统层面，真正的线程还没有创建。
2. 可运行状态，指的是线程可以分配 CPU 执行。在这种状态下，真正的操作系统线程已经被成功创建了，所以可以分配 CPU 执行。
3. 当有空闲的 CPU 时，操作系统会将其分配给一个处于可运行状态的线程，被分配到 CPU 的线程的状态就转换成了运行状态。
4. 运行状态的线程如果调用一个阻塞的 API（例如以阻塞方式读文件）或者等待某个事件（例如条件变量），那么线程的状态就会转换到休眠状态，同时释放 CPU 使用权，休眠状态的线程永远没有机会获得 CPU 使用权。当等待的事件出现了，线程就会从休眠状态转换到可运行状态。
5. 线程执行完或者出现异常就会进入终止状态，终止状态的线程不会切换到其他任何状态，进入终止状态也就意味着线程的生命周期结束了。



Java 层面的线程生命周期

1. NEW（初始化状态）
2. RUNNABLE（可运行状态+运行状态）
3. BLOCKED（阻塞状态）
4. WAITING（无时限等待）
5. TIMED_WAITING（有时限等待）
6. TERMINATED（终止状态）

![image-20211229001246243](/Users/oraliyuan/Library/Application Support/typora-user-images/image-20211229001246243.png)



![线程状态](../image/线程状态.png)



Java 线程创建的几种方式

1 **使用 Thread类或继承Thread类**

```java
Thread t = new Thread() {
    public void run() {
    // 要执行的任务
    }
};
// 启动线程
t.start();
```



2 **实现 Runnable 接口配合Thread**

```java
Runnable runnable = new Runnable() {
    public void run(){
    // 要执行的任务
    }
};
// 创建线程对象
Thread t = new Thread( runnable );
// 启动线程
t.start();
```



3 **使用有返回值的 Callable**

```java
class CallableTask implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        return new Random().nextInt();
    }
}
//创建线程池
ExecutorService service = Executors.newFixedThreadPool(10);
//提交任务，并用 Future提交返回结果
Future<Integer> future = service.submit(new CallableTask());
```



Java start源码分析

https://note.youdao.com/ynoteshare/index.html?id=4a6be81d7250f9a0e4c12e077c2f3e88&type=note&_time=1640688406458

Java 线程属于内核级线程，操作系统可以感知。用户级线程由程序来管理，操作系统感知不到。







调度机制分类：

* **协同式线程调度** ： **线程执行时间由线程本身来控制**，线程把自己的工作执行完之后，要主动通知系统切换到另外一个线程上。最大好处是实现简单，且切换操作对线程自己是可知的，没啥线程同步问题。坏处是线程执行时间不可控制，如果一个线程有问题，可能一直阻塞在那里。
* **抢占式线程调度** ： **每个线程将由系统来分配执行时间，线程的切换不由线程本身来决定**（Java中，Thread.yield()可以让出执行时间，但无法获取执行时间）。线程执行时间系统可控，也不会有一个线程导致整个进程阻塞。

Java 线程调度就是抢占式调度



Thread 常用方法：

sleep

* 调用 sleep 会让当前线程从 *Running* 进入TIMED_WAITING状态，不会释放对象锁
* 其它线程可以使用 interrupt 方法打断正在睡眠的线程，这时 sleep 方法会抛出 InterruptedException，并且会清除中断标志
* 睡眠结束后的线程未必会立刻得到执行
* sleep当传入参数为0时，和yield相同



yield：

* yield会释放CPU资源，让当前线程从 Running 进入 Runnable状态，让优先级更高（至少是相同）的线程获得执行机会，不会释放对象锁；
* 假设当前进程只有main线程，当调用yield之后，main线程会继续运行，因为没有比它优先级更高的线程；
* 具体的实现依赖于操作系统的任务调度器



join：

等待某个线程结束

```java
    public static void main(String[] sure) throws InterruptedException {

        Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("t begin");
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("t finished");
            }
        });
        long start = System.currentTimeMillis();
        t.start();
        //主线程等待线程t执行完成
        t.join();

        System.out.println("执行时间:" + (System.currentTimeMillis() - start));
        System.out.println("Main finished");
    }
```



Stop: 

强行终止一个线程，是过时方法

stop会释放对象锁，可能会造成数据不一致。

```java
public class ThreadStopDemo {

    private static Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {

        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (lock) {
                    System.out.println(Thread.currentThread().getName() + "获取锁");
                    try {
                        Thread.sleep(60000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread().getName() + "执行完成");
            }
        });
        thread.start();
        Thread.sleep(2000);
        // 停止thread，并释放锁
        thread.stop();

        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + "等待获取锁");
                synchronized (lock) {
                    System.out.println(Thread.currentThread().getName() + "获取锁");
                }
            }
        }).start();

    }
}
```



interrupt:

Java 线程中断机制：

Java没有提供一种安全、直接的方法来停止某个线程，而是提供了中断机制。中断机制是一种协作机制，也就是说通过中断并不能直接终止另一个线程，而需要被中断的线程自己处理。被中断的线程拥有完全的自主权，它既可以选择立即停止，也可以选择一段时间后停止，也可以选择压根不停止。

* interrupt()： 将线程的中断标志位设置为true，不会停止线程
* isInterrupted(): 判断当前线程的中断标志位是否为true，不会清除中断标志位
* Thread.interrupted()：判断当前线程的中断标志位是否为true，并清除中断标志位，重置为fasle



```java
public class ThreadInterruptTest {

    static int i = 0;

    public static void main(String[] args)  {
        System.out.println("begin");
        Thread t1 = new Thread(new Runnable() {
            @Override
            public  void run() {
                while (true) {
                    i++;
                    System.out.println(i);
                    try {
                        Thread.sleep(10000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    //Thread.interrupted()  清除中断标志位
                    //Thread.currentThread().isInterrupted() 不会清除中断标志位
                    if (Thread.interrupted()) {
                        System.out.println("=========");
                    }
                    if(i==10){
                        break;
                    }

                }
            }
        });

        t1.start();
        //不会停止线程t1,只会设置一个中断标志位 flag=true
        t1.interrupt();

    }
}
```



安全停止线程：

```java
public class StopThread implements Runnable {

    @Override
    public void run() {
        int count = 0;
        while (!Thread.currentThread().isInterrupted() && count < 1000) {
            System.out.println("count = " + count++);
        }
        System.out.println("线程停止： stop thread");
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new StopThread());
        thread.start();
        Thread.sleep(5);
        thread.interrupt();
    }
}
```



java 线程间通信

**volatile**

```java
public class VolatileDemo {

    private static volatile boolean flag = true;

    public static void main(String[] args) {

        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    if (flag) {
                        System.out.println("trun on");
                        flag = false;
                    }
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    if (!flag) {
                        System.out.println("trun off");
                        flag = true;
                    }
                }
            }
        }).start();
    }
}
```



notify & wait 

等待唤醒机制： 等待唤醒机制可以基于wait和notify方法来实现，在一个线程内调用该线程锁对象的wait方法，线程将进入等待队列进行等待直到被唤醒。

* wait 会释放锁
* notify 不能指定唤醒那个线程

wait&notify 实现的等待唤醒机制：

```java
public class WaitDemo {
    private static Object lock = new Object();
    private static boolean flag = true;

    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (lock) {
                    while (flag) {
                        try {
                            System.out.println("wait start .......");
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                    System.out.println("wait end ....... ");
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                if (flag) {
                    synchronized (lock) {
                        if (flag) {
                            lock.notify();
                            System.out.println("notify .......");
                            flag = false;
                        }

                    }
                }
            }
        }).start();
    }
}
```



park & unpack 实现的等待唤醒机制：

LockSupport是JDK中用来实现线程阻塞和唤醒的工具，线程调用park则等待“许可”，调用unpark则为指定线程提供“许可”。使用它可以在任何场合使线程阻塞，可以指定任何线程进行唤醒，并且不用担心阻塞和唤醒操作的顺序，但要注意连续多次唤醒的效果和一次唤醒是一样的。

```java
public class LockSupportTest {

    public static void main(String[] args) {
        Thread parkThread = new Thread(new ParkThread());
        parkThread.start();

        System.out.println("唤醒parkThread");
        LockSupport.unpark(parkThread);
    }

    static class ParkThread implements Runnable{

        @Override
        public void run() {
            System.out.println("ParkThread开始执行");
            LockSupport.park();
            System.out.println("ParkThread执行完成");
        }
    }
}
```

