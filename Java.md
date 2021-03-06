# Java

## 多线程

### 线程状态

1. **初始(NEW)**：新创建了一个线程对象，但还没有调用start()方法。
2. **运行(RUNNABLE)**：Java线程中将就绪（ready）和运行中（running）两种状态笼统的称为“运行”。线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取CPU的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得CPU时间片后变为运行中状态（running）。
3. **阻塞(BLOCKED)**：表示线程阻塞于锁。
4. **等待(WAITING)**：进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）。
5. **超时等待(TIMED_WAITING)**：该状态不同于WAITING，它可以在指定的时间后自行返回。
6. **终止(TERMINATED)**：表示该线程已经执行完毕。

![线程状态](http://oss.jankinwu.com/img/2018070117435683.jpg)

![java-thread](http://oss.jankinwu.com/img/java-thread.jpg)

### 线程的优先级

每一个 Java 线程都有一个优先级，这样有助于操作系统确定线程的调度顺序。

Java 线程的优先级是一个整数，其取值范围是 1 （Thread.MIN_PRIORITY ） - 10 （Thread.MAX_PRIORITY ）。

默认情况下，每一个线程都会分配一个优先级 NORM_PRIORITY（5）。

具有较高优先级的线程对程序更重要，并且应该在低优先级的线程之前分配处理器资源。但是，线程优先级不能保证线程执行的顺序，而且非常依赖于平台。

### 创建一个线程

Java 提供了三种创建线程的方法：

- 通过实现 Runnable 接口；
- 通过继承 Thread 类本身；
- 通过 Callable 和 Future 创建线程。

### 同步锁

非静态方法使用 synchronized 修饰的写法，修饰实例方法时，锁定的是当前对象：

```java
public synchronized void test(){
    // TODO
}
```

代码块使用 synchronized 修饰的写法，使用代码块，如果传入的参数是 this，那么锁定的也是当前的对象：

```java
public void test(){
    synchronized (this) {
        // TODO
    }
}
```

#### CopyOnWriteArrayList

下面从“动态数组”和“线程安全”两个方面进一步对CopyOnWriteArrayList的原理进行说明。

1. **CopyOnWriteArrayList的“动态数组”机制** -- 它内部有个“volatile数组”(array)来保持数据。在“添加/修改/删除”数据时，都会新建一个数组，并将更新后的数据拷贝到新建的数组中，最后再将该数组赋值给“volatile数组”。这就是它叫做CopyOnWriteArrayList的原因！CopyOnWriteArrayList就是通过这种方式实现的动态数组；不过正由于它在“添加/修改/删除”数据时，都会新建数组，所以涉及到修改数据的操作，CopyOnWriteArrayList效率很低；但是单单只是进行遍历查找的话，效率比较高。
2. **CopyOnWriteArrayList的“线程安全”机制** -- 是通过volatile和互斥锁来实现的。(01) CopyOnWriteArrayList是通过“volatile数组”来保存数据的。一个线程读取volatile数组时，总能看到其它线程对该volatile变量最后的写入；就这样，通过volatile提供了“读取到的数据总是最新的”这个机制的保证。(02) CopyOnWriteArrayList通过互斥锁来保护数据。在“添加/修改/删除”数据时，会先“获取互斥锁”，再修改完毕之后，先将数据更新到“volatile数组”中，然后再“释放互斥锁”；这样，就达到了保护数据的目的。 

#### Lock 锁

**Lock和syncronized的区别**
+ synchronized是Java语言的关键字。Lock是一个类。
+ synchronized不需要用户去手动释放锁，发生异常或者线程结束时自动释放锁;Lock则必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象。
+ lock可以配置公平策略,实现线程按照先后顺序获取锁。
+ 提供了trylock()方法 可以试图获取锁，获取到或获取不到时，返回不同的返回值 让程序可以灵活处理。
+ lock()和unlock()可以在不同的方法中执行,可以实现同一个线程在上一个方法中lock()在后续的其他方法中unlock(),比syncronized灵活的多。
+ Lock只有代码块锁，synchronized有代码块锁和方法锁
+ 使用Lock锁，JVM将花费较少的时间来调度线程，性能更好。并且具有更好的扩展性（提供更多的子类）
+ 优先使用顺序：Lock>同步代码块（已经进入了方法体，分配了相应资源）>同步方法（在方法体之外）

### 线程通信

| 方法名             | 作用                                                         |
| ------------------ | ------------------------------------------------------------ |
| wait()             | 表示线程一直等待，直到其他线程通知，与seep不同，会释放锁     |
| wait(long timeout) | 指定等待的亳秒数                                             |
| notify()           | 唤醒一个处于等待状态的线程                                   |
| notifyAll()        | 唤醒同一个对象上所有调用wat（）方法的线程，优先级别高的线程优先调度 |

注意：均是 Object类的方法，都只能在同步方法或者同步代码块中使用，否则会抛出异常 IllegaMonitorState Exception

**管程法**

+ 生产者∶负责生产数据的模块（可能是方法，对象，线程，进程）；
+ 消费者：负责处理数据的模块（可能是方法，对象，线程，进程）；
+ 缓冲区：消费者不能直接使用生产者的数据，他们之间有个“缓冲区”，生产者将生产好的数据放入缓冲区，消费者从缓冲区拿出数据

**信号灯法（通过标志位）**

+ 标志位为 true，表示产品为空
+ 当标志位为 true，生产者生产产品，并将标志位设为false，通知消费者消费
+ 当标志位为 false，消费者消费产品，并将标志位设为true，通知生产者生产

## 线程池

**Java通过Executors提供四种线程池，分别为**

1. newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。
2. newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
3. newScheduledThreadPool 创建一个可定期或者延时执行任务的定长线程池，支持定时及周期性任务执行。 
4. newCachedThreadPool 创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。 

**使用线程池的好处**

+ 降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
+ 提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
+ 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

## JVM

![java架构图](http://oss.jankinwu.com/img/java%E6%9E%B6%E6%9E%84%E5%9B%BE.png)

### 类加载器

> 类加载步骤

1. 类加载器收到类加载的请求
2. 将这个请求向上委托给父类加载器去完成，一直向上委托，直到启动类加载器（双亲委派机制）
3. 启动加载器检查是否能够加载当前这个类，能加载就结束，使用当前的加载器，否则，抛出异常，通知子加载器进行加载
4. 重复步骤3

### 沙箱安全机制

![20181022102746857](http://oss.jankinwu.com/img/20181022102746857.png)

**组成沙箱的基本组件：**

- `字节码校验器`（bytecode verifier）：确保Java类文件遵循Java语言规范。这样可以帮助Java程序实现内存保护。但并不是所有的类文件都会经过字节码校验，比如核心类。

- `类装载器`（class loader）：其中类装载器在3个方面对Java沙箱起作用
  - 它防止恶意代码去干涉善意的代码；
  - 它守护了被信任的类库边界；
  - 它将代码归入保护域，确定了代码可以进行哪些操作。

  虚拟机为不同的类加载器载入的类提供不同的命名空间，命名空间由一系列唯一的名称组成，每一个被装载的类将有一个名字，这个命名空间是由Java虚拟机为每一个类装载器维护的，它们互相之间甚至不可见。

  类装载器采用的机制是`双亲委派模式`。

1. 从最内层JVM自带类加载器开始加载，外层恶意同名类得不到加载从而无法使用；
2. 由于严格通过包来区分了访问域，外层恶意的类通过内置代码也无法获得权限访问到内层类，破坏代码就自然无法生效。

- `存取控制器`（access controller）：存取控制器可以控制核心API对操作系统的存取权限，而这个控制的策略设定，可以由用户指定。

- `安全管理器`（security manager）：是核心API和操作系统之间的主要接口。实现权限控制，比存取控制器优先级高。

- `安全软件包`（security package）：java.security下的类和扩展包下的类，允许用户为自己的应用增加新的安全特性，包括：
  - 安全提供者
  - 消息摘要
  - 数字签名
  - 加密
  - 鉴别

### JNI

JNI是Java Native Interface的缩写，通过使用 Java本地接口书写程序，可以确保代码在不同的平台上方便移植。

> native 关键字

​		native关键字说明其修饰的方法是一个原生态方法，方法对应的实现不是在当前文件，而是在用其他语言（如C和C++）实现的文件中。Java语言本身不能对操作系统底层进行访问和操作，但是可以通过JNI接口调用其他语言来实现对底层的访问。JNI是Java Native Interface的缩写，它提供了若干的API实现了Java和其他语言的通信（主要C&C++）。从Java1.1开始，JNI标准成为java平台的一部分，它允许Java代码和其他语言写的代码进行交互。JNI一开始是为了本地已编译语言，尤其是C和C++而设计的，但是它并不妨碍你使用其他编程语言，只要调用约定受支持就可以了。

### 方法区

​		方法区主要存放已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据（比如spring 使用IOC或者AOP创建bean时，或者使用cglib，反射的形式动态生成class信息等）。

### 栈

![栈帧结构](http://oss.jankinwu.com/img/17891124-f2d9ec2a1e1bf4d0.png)

​		每当启动一个新线程时，Java虚拟机都会为它分配一个Java栈。Java栈以帧为单位保存线程的运行状态。虚拟机只会直接对Java栈执行两种操作：以帧为单位的压栈和出栈。

​		某个线程正在执行的方法被称为该线程的当前方法，当前方法使用的栈帧称为当前帧，当前方法所属的类称为当前类，当前类的常量池称为当前常量池。在线程执行一个方法时，它会跟踪当前类和当前常量池。此外，当虚拟机遇到栈内操作指令时，它对当前帧内数据执行操作。

​		每当线程调用一个Java方法时，虚拟机都会在该线程的Java栈中压入一个新帧。而这个新帧自然就成为了当前帧。在执行这个方法时，它使用这个帧来存储参数、局部变量、中间运算结果等数据。
　　Java方法可以以两种方式完成。一种通过return返回的，称为正常返回；一种是通过抛出异常而异常终止的。不管以哪种方式返回，虚拟机都会将当前帧弹出Java栈然后释放掉，这样上一个方法的帧就成为当前帧了。

　　Java帧上的所有数据都是此线程私有的。任何线程都不能访问另一个线程的栈数据，因此我们不需要考虑多线程情况下栈数据的访问同步问题。当一个线程调用一个方法时，方法的的局部变量保存在调用线程Java栈的帧中。只有一个线程能总是访问那些局部变量，即调用方法的线程。

​		主管程序的运行，生命周期和线程同步；线程结束，栈内存也就是释放，对于栈来说，不存在垃圾回收问题旦线程结束，栈就Over！

栈里存放的对象：8大基本类型+对象引用+实例的方法

### 堆

**堆内存划分：**

![jvm堆内存划分](http://oss.jankinwu.com/img/jvm%E5%A0%86%E5%86%85%E5%AD%98%E5%88%92%E5%88%86.jpg)

![从垃圾回收角度划分堆](http://oss.jankinwu.com/img/32fa828ba61ea8d336768dd0e1e49547271f5882.jpeg)

​		Java虚拟机将堆内存划分为新生代、老年代和永久代，永久代是HotSpot虚拟机特有的概念（JDK1.8之后为metaspace替代永久代），它采用永久代的方式来实现方法区，其他的虚拟机实现没有这一概念，而且HotSpot也有取消永久代的趋势，在JDK 1.7中HotSpot已经开始了“去永久化”，把原本放在永久代的字符串常量池移出。永久代主要存放常量、类信息、静态变量等数据，与垃圾回收关系不大，新生代和老年代是垃圾回收的主要区域。

**新生代（Young Generation）**

​		新生成的对象优先存放在新生代中，新生代对象朝生夕死，存活率很低，在新生代中，常规应用进行一次垃圾收集一般可以回收70% ~ 95% 的空间，回收效率很高。
　　HotSpot将新生代划分为三块，一块较大的Eden（伊甸）空间和两块较小的Survivor（幸存者）空间，默认比例为8：1：1。划分的目的是因为HotSpot采用复制算法来回收新生代，设置这个比例是为了充分利用内存空间，减少浪费。新生成的对象在Eden区分配（大对象除外，大对象直接进入老年代），当Eden区没有足够的空间进行分配时，虚拟机将发起一次Minor GC。
　　GC开始时，对象只会存在于Eden区和From Survivor区，To Survivor区是空的（作为保留区域）。GC进行时，Eden区中所有存活的对象都会被复制到To Survivor区，而在From Survivor区中，仍存活的对象会根据它们的年龄值决定去向，年龄值达到年龄阀值（默认为15，新生代中的对象每熬过一轮垃圾回收，年龄值就加1，GC分代年龄存储在对象的header中）的对象会被移到老年代中，没有达到阈值的对象会被复制到To Survivor区。接着清空Eden区和From Survivor区，新生代中存活的对象都在To Survivor区。接着， From Survivor区和To Survivor区会交换它们的角色，也就是新的To Survivor区就是上次GC清空的From Survivor区，新的From Survivor区就是上次GC的To Survivor区，总之，不管怎样都会保证To Survivor区在一轮GC后是空的。GC时当To Survivor区没有足够的空间存放上一次新生代收集下来的存活对象时，需要依赖老年代进行分配担保，将这些对象存放在老年代中。

**老年代（Old Generationn）**

　　在新生代中经历了多次（具体看虚拟机配置的阈值）GC后仍然存活下来的对象会进入老年代中。老年代中的对象生命周期较长，存活率比较高，在老年代中进行GC的频率相对而言较低，而且回收的速度也比较慢。

**永久代（Permanent Generationn）**

　　永久代存储类信息、常量、静态变量、即时编译器编译后的代码等数据，对这一区域而言，Java虚拟机规范指出可以不进行垃圾收集，一般而言不会进行垃圾回收。

​		永久代中出现OOM(OutOfMemory)情况：一个启动类，加载了大量第三方jar包、Tomcat部署了大量应用、大量动态生成的反射类。

```java
// 返回jvm试图使用的最大内存(单位：字节)
// JVM最大分配的内存由-Xmx指定，默认是物理内存的1/4
Runtime.getRuntime().maxMemory();
// 返回jvm的总内存(单位：字节) 
// JVM初始分配的内存由-Xms指定，默认是物理内存的1/64
Runtime.getRuntime().totalMemory();
// -XX:+PrintGCDetails // 打印垃圾回收详细信息
// -XX:+HeapDumpOnOutOfMemoryError // 当JVM发生OOM时，自动生成DUMP文件
```



