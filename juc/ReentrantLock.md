





## 1. 概念

ReentrantLock是一种基于AQS框架的应用实现，是JDK中的一种线程并发访问的同步手段，它的功能类似于synchronized是一种互斥锁，可以保证线程安全。



对比synchronized ： 

* 可中断 
* 可以设置超时时间
* 可以设置为公平锁 
* 支持多个条件变量
* 它们都支持重入



synchronized和ReentrantLock的区别：

* synchronized是JVM层次的锁实现，ReentrantLock是JDK层次的锁实现；
* synchronized的锁状态是无法在代码中直接判断的，但是ReentrantLock可以通过ReentrantLock#isLocked判断；
* synchronized是非公平锁，ReentrantLock是可以是公平也可以是非公平的；
* synchronized是不可以被中断的，而ReentrantLock#lockInterruptibly方法是可以被中断的；
* 在发生异常时synchronized会自动释放锁，而ReentrantLock需要开发者在finally块中显示释放锁；
* ReentrantLock获取锁的形式有多种：如立即返回是否成功的tryLock(),以及等待指定时长的获取，更加灵活；
* synchronized在特定的情况下对于已经在等待的线程是后来的线程先获得锁（回顾一下sychronized的唤醒策略），而ReentrantLock对于已经在等待的线程是先来的线程先获得锁；











## 2. 使用



### 2.1 可重入

```java
public class ReentrantLockDemo2 {

    public static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        method1();
    }


    public static void method1() {
        lock.lock();
        try {
            System.out.println("execute method1");
            method2();
        } finally {
            lock.unlock();
        }
    }
    public static void method2() {
        lock.lock();
        try {
            System.out.println("execute method2");
            method3();
        } finally {
            lock.unlock();
        }
    }
    public static void method3() {
        lock.lock();
        try {
            System.out.println("execute method3");
        } finally {
            lock.unlock();
        }
    }
}
```





### 2.2 可中断

* 中断机制可以优雅的停止线程

* 一个线程做一些事情影响另一个线程，比如一个线程把某件事情做了，另一个线程不需要做这件事了，那么就可以利用这种中断机制来实现

  

```java
public class ReentrantLockDemo3 {

    public static void main(String[] args) throws InterruptedException {
        ReentrantLock lock = new ReentrantLock();

        Thread t1 = new Thread(() -> {

            System.out.println("t1启动...");

            try {
                lock.lockInterruptibly();
                try {
                    System.out.println("t1获得了锁");
                } finally {
                    lock.unlock();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
                System.out.println("t1等锁的过程中被中断");
            }

        }, "t1");


        lock.lock();
        try {
            System.out.println("main线程获得了锁");
            t1.start();
            //先让线程t1执行
            Thread.sleep(1000);

            t1.interrupt();
            System.out.println("线程t1执行中断");
        } finally {
            lock.unlock();
        }
    }
}

```





### 2.3 锁超时

```java
public class ReentrantLockDemo4 {

    public static void main(String[] args) throws InterruptedException {
        ReentrantLock lock = new ReentrantLock();

        Thread t1 = new Thread(() -> {

            System.out.println("t1启动...");
            // 注意： 即使是设置的公平锁，此方法也会立即返回获取锁成功或失败，公平策略不生效
//            if (!lock.tryLock()) {
//                System.out.println("t1获取锁失败，立即返回false");
//                return;
//            }

            //超时
            try {
                if (!lock.tryLock(1, TimeUnit.SECONDS)) {
                    System.out.println("等待 1s 后获取锁失败，返回");
                    return;
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
                return;
            }

            try {
                System.out.println("t1获得了锁");
            } finally {
                lock.unlock();
            }

        }, "t1");


        lock.lock();
        try {
            System.out.println("main线程获得了锁");
            t1.start();
            //先让线程t1执行
            Thread.sleep(2000);
        } finally {
            lock.unlock();
        }
    }
}
```





### 2.4 公平锁

```java
public class ReentrantLockDemo5 {

    public static void main(String[] args) throws InterruptedException {
        //ReentrantLock lock = new ReentrantLock(true); //公平锁
        ReentrantLock lock = new ReentrantLock(); //非公平锁

        for (int i = 0; i < 500; i++) {
            new Thread(() -> {
                lock.lock();
                try {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + " running...");
                } finally {
                    lock.unlock();
                }
            }, "t" + i).start();
        }
        // 1s 之后去争抢锁
        Thread.sleep(1000);

        for (int i = 0; i < 500; i++) {
            new Thread(() -> {
                lock.lock();
                try {
                    System.out.println(Thread.currentThread().getName() + " running...");
                } finally {
                    lock.unlock();
                }
            }, "强行插入" + i).start();
        }
    }
}

```





### 2.5 条件变量

```java
public class ReentrantLockDemo6 {
    private static ReentrantLock lock = new ReentrantLock();
    private static Condition cigCon = lock.newCondition();
    private static Condition takeCon = lock.newCondition();

    private static boolean hashcig = false;
    private static boolean hastakeout = false;

    //送烟
    public void cigratee(){
        lock.lock();
        try {
            while(!hashcig){
                try {
                    System.out.println("没有烟，歇一会");
                    cigCon.await();
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
            System.out.println("有烟了，干活");
        }finally {
            lock.unlock();
        }
    }

    //送外卖
    public void takeout(){
        lock.lock();
        try {
            while(!hastakeout){
                try {
                    System.out.println("没有饭，歇一会");
                    takeCon.await();
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
            System.out.println("有饭了，干活");
        }finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        ReentrantLockDemo6 test = new ReentrantLockDemo6();
        new Thread(() ->{
            test.cigratee();
        }).start();

        new Thread(() -> {
            test.takeout();
        }).start();

        new Thread(() ->{
            lock.lock();
            try {
                hashcig = true;
                System.out.println("唤醒送烟的等待线程");
                cigCon.signal();
            }finally {
                lock.unlock();
            }
        },"t1").start();

        new Thread(() ->{
            lock.lock();
            try {
                hastakeout = true;
                System.out.println("唤醒送饭的等待线程");
                takeCon.signal();
            }finally {
                lock.unlock();
            }
        },"t2").start();
    }
}
```











## 3. 源码流程图



ReentrantLock 灵魂质问：

* ReentrantLock 加锁解锁的逻辑
* 公平和非公平的实现，可重入的实现
* 线程竞争锁失败入队阻塞逻辑
* 获取锁后释放锁、唤醒阻塞线程竞争锁的逻辑实现



先模拟一波三个线程竞争锁的场景

```java
public class ReentrantLockDemo {

    private static  int sum = 0;
    private static Lock lock = new ReentrantLock();

    public static void main(String[] args) throws InterruptedException {

        for (int i = 0; i < 3; i++) {
            Thread thread = new Thread(()->{
                //加锁
                lock.lock();
                try {
                    // 临界区代码
                    for (int j = 0; j < 10000; j++) {
                        sum++;
                    }
                } finally {
                    // 解锁
                    lock.unlock();
                }
            });
            thread.start();
        }

        Thread.sleep(2000);
        System.out.println(sum);
    }
}
```



![ReentrantLock 加锁逻辑](/Users/oraliyuan/Desktop/笔记/image/ReentrantLock 加锁逻辑.png)





此处体现公平和非公平锁：

![image-20211231180409597](/Users/oraliyuan/Library/Application Support/typora-user-images/image-20211231180409597.png)







![image-20211231183259982](/Users/oraliyuan/Library/Application Support/typora-user-images/image-20211231183259982.png)



解锁逻辑：

![ReentrantLock 解锁逻辑](/Users/oraliyuan/Desktop/笔记/image/ReentrantLock 解锁逻辑.png)
