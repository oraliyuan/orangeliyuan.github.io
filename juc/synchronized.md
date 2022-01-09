



临界区：

* 一个程序运行多个线程本身是没有问题的
* 问题出在多个线程访问共享资源
  * 多个线程读共享资源其实也没问题
  * 多个线程对共享资源读写操作时发生指令交错，就会出现问题

```java
//临界资源
private static int counter = 0;

//临界区
public static void increment() { 
    counter++;
}

//临界区
public static void decrement() {
    counter--;
}
```



竞态条件：

多个线程在临界区内执行，由于代码的执行序列不同而导致结果无法预测，称之为发生了竞态条件

为了避免临界区的竞态条件发生，有多种手段可以达到目的：

* 阻塞式的解决方案：synchronized，Lock 
* 非阻塞式的解决方案：原子变量





## 1 synchronized 使用

synchronized 同步块是 Java 提供的一种原子性内置锁，Java 中的每个对象都可以把它当作一个同步锁来使用，这些 Java 内置的使用者看不到的锁被称为内置锁，也叫作监视器锁。

![image-20211231012256469](/Users/oraliyuan/Library/Application Support/typora-user-images/image-20211231012256469.png)



## 2 synchronized 原理

synchronized是JVM内置锁，基于**Monitor**机制实现，依赖底层操作系统的互斥原语Mutex（互斥量），它是一个重量级锁，性能较低。

JVM内置锁在1.5之后版本做了重大的优化，如锁粗化（Lock Coarsening）、锁消除（Lock Elimination）、轻量级锁（Lightweight Locking）、偏向锁（Biased Locking）、自适应自旋（Adaptive Spinning）等技术来减少锁操作的开销，内置锁的并发性能已经基本与Lock持平



同步方法是通过方法中的access_flags中设置ACC_SYNCHRONIZED标志来实现；同步代码块是通过monitorenter和monitorexit来实现。两个指令的执行是JVM通过调用操作系统的互斥原语mutex来实现，被阻塞的线程会被挂起、等待重新调度，会导致“用户态和内核态”两个态之间来回切换，对性能有较大影响。



### 2.1 Monitor (管程 / 监视器)

Monitor，直译为“监视器”，而操作系统领域一般翻译为“管程”。管程是指管理共享变量以及对共享变量操作的过程，让它们支持并发。

synchronized关键字和wait()、notify()、notifyAll()这三个方法是Java中实现管程技术的组成部分。

synchronized 就是Java 对操作系统管程的一种实现。

JVM 中ObjectMonitor 对象结构如下：

```c++
ObjectMonitor() {
    _header       = NULL; //对象头  markOop
    _count        = 0;  
    _waiters      = 0,   
    _recursions   = 0;   // 锁的重入次数 
    _object       = NULL;  //存储锁对象
    _owner        = NULL;  // 标识拥有该monitor的线程（当前获取锁的线程） 
    _WaitSet      = NULL;  // 等待线程（调用wait）组成的双向循环链表，_WaitSet是第一个节点
    _WaitSetLock  = 0 ;    
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ; //多线程竞争锁会先存到这个单向链表中 （FILO栈结构）
    FreeNext      = NULL ;
    _EntryList    = NULL ; //存放在进入或重新进入时被阻塞(blocked)的线程 (也是存竞争锁失败的线程)
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
    _previous_owner_tid = 0;
}
```





#### 2.1.1 MESA 模型

一种管程的具体模型。在管程的发展史上，先后出现过三种不同的管程模型，分别是Hasen模型、Hoare模型和MESA模型。现在正在广泛使用的是MESA模型。

![image-20211231013017592](/Users/oraliyuan/Library/Application Support/typora-user-images/image-20211231013017592.png)

管程在JVM中的具体实现的流程图：

在获取锁时，是将当前线程插入到cxq的头部，而释放锁时：如果EntryList为空，则将cxq中的元素按原有顺序插入到EntryList，并唤醒第一个线程，也就是当EntryList为空时，是后来的线程先获取锁（所以synchronized 是非公平锁）。_EntryList不为空，直接从_EntryList中唤醒线程。

![image-20211231013041982](/Users/oraliyuan/Library/Application Support/typora-user-images/image-20211231013041982.png)



### 2.2 与锁相关的对象头Mark word

**Mark Word的结构**

* hash： 保存对象的哈希码。运行期间调用System.identityHashCode()来计算，延迟计算，并把结果赋值到这里。
* age： 保存对象的分代年龄。表示对象被GC的次数，当该次数到达阈值的时候，对象就会转移到老年代。
* biased_lock： 偏向锁标识位。由于无锁和偏向锁的锁标识都是 01，没办法区分，这里引入一位的偏向锁标识位。
* lock： 锁状态标识位。区分锁状态，比如11时表示对象待GC回收状态, 只有最后2位锁标识(11)有效。
* JavaThread： 保存持有偏向锁的线程ID。偏向模式的时候，当某个线程持有对象的时候，对象这里就会被置为该线程的ID。 在后面的操作中，就无需再进行尝试获取锁的动作。这个线程ID并不是JVM分配的线程ID号，和Java Thread中的ID是两个概念。
* epoch： 保存偏向时间戳。偏向锁在CAS锁操作过程中，偏向性标识，表示对象更偏向哪个锁。
* ptr_to_lock_record：轻量级锁状态下，指向栈中锁记录的指针。当锁获取是无竞争时，JVM使用原子操作而不是OS互斥，这种技术称为轻量级锁定。在轻量级锁定的情况下，JVM通过CAS操作在对象的Mark Word中设置指向锁记录的指针。
* ptr_to_heavyweight_monitor：重量级锁状态下，指向对象监视器Monitor的指针。如果两个不同的线程同时在同一个对象上竞争，则必须将轻量级锁定升级到Monitor以管理等待的线程。在重量级锁定的情况下，JVM在对象的ptr_to_heavyweight_monitor设置指向Monitor的指针



32 位 JVM下的对象头结构：

![image-20211231013751790](/Users/oraliyuan/Library/Application Support/typora-user-images/image-20211231013751790.png)



64 位JVM下的对象头结构：

![image-20211231013829484](/Users/oraliyuan/Library/Application Support/typora-user-images/image-20211231013829484.png)



锁相关标识位：

![image-20211231013915698](/Users/oraliyuan/Library/Application Support/typora-user-images/image-20211231013915698.png)





### 2.2.1 锁升级

##### 2.2.1.1 偏向锁

偏向锁是一种针对加锁操作的优化手段，经过研究发现，在大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，因此为了消除数据在无竞争情况下锁重入（CAS操作）的开销而引入偏向锁。对于没有锁竞争的场合，偏向锁有很好的优化效果。

当JVM启用了偏向锁模式（jdk6默认开启），新创建对象的Mark Word中的Thread Id为0，说明此时处于可偏向但未偏向任何线程，也叫做匿名偏向状态(anonymously biased)。



##### 2.2.1.2 偏向锁延迟偏向

偏向锁模式存在偏向锁延迟机制：HotSpot 虚拟机在启动后有个 4s 的延迟才会对每个新建的对象开启偏向锁模式。JVM启动时会进行一系列的复杂活动，比如装载配置，系统类初始化等等。在这个过程中会使用大量synchronized关键字对对象加锁，且这些锁大多数都不是偏向锁。为了减少初始化时间，JVM默认延时加载偏向锁。

```java
public static void main(String[] args) throws InterruptedException {
  System.out.println(ClassLayout.parseInstance(new Object()).toPrintable());
  Thread.sleep(4000);
  System.out.println(ClassLayout.parseInstance(new Object()).toPrintable());
}
```



![image-20211231124440913](/Users/oraliyuan/Library/Application Support/typora-user-images/image-20211231124440913.png)



##### 2.2.1.3 偏向锁撤销

* 调用锁对象的obj.hashCode()或System.identityHashCode(obj)方法会导致该对象的偏向锁被撤销。因为对于一个对象，其HashCode只会生成一次并保存，偏向锁是没有地方保存hashcode的。
  * 轻量级锁会在锁记录中记录 hashCode 
  * 重量级锁会在 Monitor 中记录 hashCode
  * 当对象可偏向时，MarkWord将变成未锁定状态，并只能升级成轻量锁；
  * 当对象正处于偏向锁时，调用HashCode将使偏向锁强制升级成重量锁。
*  偏向锁状态执行obj.notify() 会升级为轻量级锁，调用obj.wait(timeout) 会升级为重量级锁



##### 2.2.1.4 轻量级锁

倘若偏向锁失败，虚拟机并不会立即升级为重量级锁，它还会尝试使用一种称为轻量级锁的优化手段，此时Mark Word 的结构也变为轻量级锁的结构。**轻量级锁所适应的场景是线程交替执行同步块的场合，**如果存在同一时间多个线程访问同一把锁的场合，就会导致轻量级锁膨胀为重量级锁。

轻微竞争时轻量级锁会进行一次CAS尝试替换对象头中的指针，如果失败那么就会进行锁膨胀。

轻量级锁不存在自旋。



##### 2.2.1.5 重量级锁

重量级锁使用JVM 指令monitorenter、monitorexit 来实现，底层是由操作系统来保证原子性的。

重量级锁适应于多线程竞争激烈的场景，膨胀期间创建一个monitor对象，膨胀期间可能会进行CAS自旋，也可能是一定次数的CAS自旋，如果一直失败那么就将该线程进行阻塞。



![image-20211231133245390](/Users/oraliyuan/Library/Application Support/typora-user-images/image-20211231133245390.png)



#### 2.2.2 锁优化

##### 2.2.2.1 偏向锁批量重偏向&批量撤销

从偏向锁的加锁解锁过程中可看出，当只有一个线程反复进入同步块时，偏向锁带来的性能开销基本可以忽略，但是当有其他线程尝试获得锁时，就需要等到safe point时，再将偏向锁撤销为无锁状态或升级为轻量级，会消耗一定的性能，所以在多线程竞争频繁的情况下，偏向锁不仅不能提高性能，还会导致性能下降。于是，就有了批量重偏向与批量撤销的机制。



以class为单位，为每个class维护一个偏向锁撤销计数器，每一次该class的对象发生偏向撤销操作时，该计数器+1，当这个值达到重偏向阈值（默认20）时，JVM就认为该class的偏向锁有问题，因此会进行批量重偏向。

每个class对象会有一个对应的epoch字段，每个处于偏向锁状态对象的Mark Word中也有该字段，其初始值为创建该对象时class中的epoch的值。每次发生批量重偏向时，就将该值+1，同时遍历JVM中所有线程的栈，找到该class所有正处于加锁状态的偏向锁，将其epoch字段改为新值。下次获得锁时，发现当前对象的epoch值和class的epoch不相等，那就算当前已经偏向了其他线程，也不会执行撤销操作，而是直接通过CAS操作将其Mark Word的Thread Id 改成当前线程Id。

当达到重偏向阈值（默认20）后，假设该class计数器继续增长，当其达到批量撤销的阈值后（默认40），JVM就认为该class的使用场景存在多线程竞争，会标记该class为不可偏向，之后，对于该class的锁，直接走轻量级锁的逻辑。

批量重偏向（bulk rebias）机制是为了解决：一个线程创建了大量对象并执行了初始的同步操作，后来另一个线程也来将这些对象作为锁对象进行操作，这样会导致大量的偏向锁撤销操作。 

批量撤销（bulk revoke）机制是为了解决：在明显多线程竞争剧烈的场景下使用偏向锁是不合适的。



##### 2.2.2.2 自旋优化

重量级锁竞争的时候，还可以使用自旋来进行优化，如果当前线程自旋成功（即这时候持锁线程已经退出了同步块，释放了锁），这时当前线程就可以避免阻塞。

这种自选自适应是智能的，也就是尽量合理的进行自旋，减少空转。自旋的目的是为了减少线程挂起的次数，尽量避免直接挂起线程。



##### 2.2.2.3 锁粗化

假设一系列的连续操作都会对同一个对象反复加锁及解锁，甚至加锁操作是出现在循环体中的，即使没有出现线程竞争，频繁地进行互斥同步操作也会导致不必要的性能损耗。如果JVM检测到有一连串零碎的操作都是对同一对象的加锁，将会扩大加锁同步的范围（即锁粗化）到整个操作序列的外部。

```java
StringBuffer buffer = new StringBuffer();
/**
 * 锁粗化
 */
public void append(){
    buffer.append("aaa").append(" bbb").append(" ccc");
}
```



##### 2.2.2.4 锁消除

锁消除即删除不必要的加锁操作。锁消除是Java虚拟机在JIT编译期间，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁，通过锁消除，可以节省毫无意义的请求锁时间。



锁消除的判断与逃逸分析：

逃逸分析，是一种可以有效减少Java 程序中同步负载和内存堆分配压力的跨函数全局数据流分析算法。通过逃逸分析，Java Hotspot编译器能够分析出一个新的对象的引用的使用范围从而决定是否要将这个对象分配到堆上。逃逸分析的基本行为就是分析对象动态作用域。

* 方法逃逸，对象是否会逃出当前方法
* 线程逃逸，对象是否会逃出当前线程







