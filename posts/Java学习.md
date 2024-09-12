---
categories: []
date: '2024-06-25'
description: Java学习的一些总结
tags:
- Java
title: Java学习
updated: '2024-09-12T15:02:52.855+08:00'
---
# Java 学习

[JavaGuide](https://javaguide.cn/home.html)

- 函数式接口 Function，Function<Element, C> 表示接受类型为 Element 的参数，并返回类型为 C 的结果
  - Consumer
  - Supplier
  - Predicate
  - Function
- Vector 与 ArrayList 都实现了 List 接口，Vector 是同步的线程安全类，可以考虑使用 CopyOnWriteArrayList
- Optional 用来解决 NullPointerExceptions
  - Optional.empty()
  - Optional.of(name) name 需要非 null
  - Optional.ofNullable(name)
  - isPresent() / isEmpty() / ifPresent()
  - get() / orElse() / orElseGet()
  - map() 如果有值，则对其执行调用 mapping 函数得到返回值，返回一个 Optional
- Stream
  - forEach()
  - map()
  - filter()
  - limit()
  - sorted()
  - anyMatch()
- String.valueOf() 将参数转换为字符串
- Objects.equals() 比较相等可以解决 NullPointerExceptions

## Collections

- 线程安全的List
  - Collections.synchronizedList()，当遍历返回列表时，必须手动对其进行同步
  - CopyOnWriteArrayList，读写分离，放弃数据实时性，适用于读多写少
- 线程安全的Set
  - Collections.newSetFromMap(new ConcurrentHashMap<Integer, Boolean>())
  - CopyOnWriteArraySet
- 线程安全的Map
  - Collections.synchronizedMap()
  - ConcurrentHashMap，分段锁
- Queue
  - ArrayBlockingQueue/LinkedBlockingQueue，阻塞队列
  - ConcurrentLinkedQueue，非阻塞队列，CAS+自旋操作，读多写少
- Vector、Hashtable，使用synchronized修饰方法，效率低下，复合操作不是线程安全的

## 多线程，并发，异步

继承 Thread类，覆写 run() 或者在创建 Thread 时传入 Runnable 实例来执行线程代码

volatile作用：

- 防止指令重排
- 确保线程内存中的对象和主线程内对象同步，即确保线程内存中对象对其他线程的可见性（可见性：总是能看到任意线程对这个volatile变量最后的写入）
- volatile关键字不能当锁用
- volatile的复合操作（例如自增）不是原子的

### 锁相关

- 乐观锁适合读多，悲观锁适合写多
- CAS：无锁算法，比较内存与预期值是否一致，一致则更新；用于实现乐观锁，例如AtomicInteger通过自旋CAS实现
  - CAS存在ABA问题，可以添加版本号来解决
- 公平锁：每个线程获取锁的顺序是按照线程访问锁的先后顺序获取的，最前面的线程总是最先获取到锁。
- 非公平锁：每个线程获取锁的顺序是随机的，并不会遵循先来先得的规则，所有线程会竞争获取锁。

Java中的锁：

`synchronized`通过Monitor来实现线程同步，Monitor依赖于底层的操作系统的Mutex Lock实现

1. 无锁
2. 偏向锁：一段同步代码一直被一个线程所访问，那么该线程会自动获取锁，降低获取锁的代价；锁的状态存储在对象头中的Mark Word，里面存储锁偏向的线程ID
3. 轻量级锁：当锁是偏向锁的时候，被另外的线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞
4. 重量级锁

`ReentrantLock`可重入锁：可以在嵌套调用时自动获得锁，避免死锁，互斥锁

`ReentrantReadWriteLock`：只在写时阻塞，适合读多写少

`stampedLock`：乐观读，悲观写，在获取锁时返回一个标识符来判断

### ThreadLocal

- ThreadLocal 维护一个线程本地变量，set，get，remove
- 内部维护了 ThreadLocalMap，用于存储 ThreadLocal 变量与各自线程的副本之间的映射关系。
- ThreadLocalMap 内部由一个 Entry 数组构成，每个 Entry 包含一个 ThreadLocal 变量和对应的线程本地副本值，计算 ThreadLocal 的哈希值然后存储到对应位置。
- Entry 继承了 WeakReference 防止内存泄漏

### CompletableFuture

- `supplyAsync`：立即启动一个异步计算
- `runAsync`：执行一个没有返回值的异步任务
- `.get()`：阻塞的获取结果
- `.thenApply()`与`.thenAccept()`：处理计算结果
- `allOf`与`anyOf`：组合
- `.handle()`与`.exceptionally()`：处理异常

### 线程池

- `Runnable`不会返回结果或抛异常，`Callable`可以
- `execute()`无返回值，`submit()`返回一个`Future`
- `shutdown()`不再接受新任务，会等待队列中任务执行完毕，`shutdownNow()`终止正在运行的任务，返回正在等待执行的 List

#### ThreadPoolExecutor 参数

- corePoolSize，核心线程数量
- maximumPoolSize，最大线程数
- keepAliveTime，当线程数大于核心线程数时，多余的空闲线程存活的最长时间
- unit，时间单位
- workQueue，任务队列
- threadFactory，线程工厂
- handler，拒绝策略

#### ThreadPoolExecutor 拒绝策略

`ThreadPoolExecutor`中的拒绝策略：

- `AbortPolicy`，抛出`RejectedExecutionException`来拒绝新任务的处理
- `CallerRunsPolicy`，调用执行自己的线程运行任务，如果执行程序已关闭，则会丢弃该任务
- `DiscardPolicy`，丢弃新任务
- `DiscardOldestPolicy`，丢弃最早的未处理的任务请求

若不允许丢弃任务：

1. 增加`BlockingQueue`大小，调整`maximumPoolSize`
2. 任务持久化，使用MySQL、Redis、消息队列等

#### 线程池常用阻塞队列

- `LinkedBlockingQueue`无界队列，容量为 `Integer.MAX_VALUE`：`FixedThreadPool` 和 `SingleThreadExector`
- `SynchronousQueue`同步队列，`CachedThreadPool`，没有容量，不存储元素
- `DelayedWorkQueue`延迟阻塞队列，`ScheduledThreadPool`和`SingleThreadScheduledExecutor`，按照延迟的时间长短对任务进行排序，添加元素满了之后会自动扩容原来容量的 1/2

#### 常用内置线程池

- `FixedThreadPool`，核心线程数和最大线程数相等，不会拒绝任务，可能OOM
- `SingleThreadExecutor`，核心线程数和最大线程数都是1
- `CachedThreadPool`，`corePoolSize`为0，`maximumPoolSize`为 `Integer.MAX.VALUE`，可能会创建大量线程导致OOM
- `ScheduledThreadPool`

## 日志

slf4j 是日志框架的接口，logback 是实现

MDC 能在多线程环境下进行日志调用链路的跟踪，在 logback.xml 中配置占位符，在线程中使用 MDC.put() 与 MDC.remove()。在 LogicServer 中在 Call 前 put uid，Call 后 remove

## 强软弱虚引用

- 强引用有引用变量指向时永远不会被垃圾回收，显示赋值null会回收
- SoftReference，内存空间不足前回收，会记录上一次gc和get的时间，差值大于_max_interval会在下一次gc回收，用于实现缓存
  - 构造时可以提供ReferenceQueue，回收后会放入ReferenceQueue中，用于清除对象
- WeakReference，gc时总会回收
- PhantomReference，不影响对象的生命周期，与ReferenceQueue同时使用，会在回收时收到通知

## Spring

- `@Autowired`默认byType，type相同byName，可以通过`@Qualifier`指定名称
- `@Resource`默认byName，可以通过`name`显式指定
- 循环依赖：使用三级缓存解决，发生循环依赖则去三级缓存调用`ObjectFactory.getObject()`获取还没有初始化属性的bean

## JVM

### 类加载器

类加载器：将class文件搬到JVM，方法区（类的元数据信息，常量和静态变量等）、堆、栈、本地方法栈（native方法）、程序计数器（虚拟机字节码指令的地址）

流程：加载、连接、初始化、卸载

**双亲委派机制**：类加载器会先委派给父类加载器去完成，直到Bootstrap类加载器，只有上一级无法完成加载请求时才会尝试自己加载

### GC

TLAB：JVM为每个线程分配的一块连续的私有内存区域

年轻代(Eden Survivor[FromPlace, ToPlace], 8:1:1)、老年代，toPlace的survivor区域是空的。非堆内存为永久代。

Eden满了会先触发Minor GC，存活下来会移到Survivor0，Survivor0满了触发Minor GC，存活下来移到Survivor1，把from和to两个指针交换，这样保证了一段时间内总有一个survivor区为空且to所指向的survivor区为空。经过多次的Minor GC后仍然存活的对象（15次）会移动到老年代，占满后Full GC，期间会停止所有线程等待GC的完成，再满会OOM。

判断对象是否存活：引用计数器（循环引用）、可达性分析（Java使用）
判断对象死亡：对象没有与GC Roots相连的引用链，第一次标记被放入F-Queue队列中，第二次标记后回收

可达性算法？

### GC算法

1. 标记清除算法
   把已死亡的对象标记为空闲内存，然后记录在一个空闲列表中，new对象时在空闲列表中寻找空闲的内存分配。效率较低，内存碎片。
2. 复制算法
   将可用内存按容量划分成两等分，每次只使用其中的一块。内存缩水。
3. 标记整理算法
   标记后让所有存活的对象都向一端移动，然后直接清理掉边界以外的内存。
4. 分代收集算法
   根据各个年代的特点采用最适当的收集算法。

JDK8：Parallel Scavenge与Parallel Old，新生代复制算法，老年代标记整理，吞吐量优先。
JDK9：G1，响应速度优先，不区分新老，优先收集存活对象较少的区域

# 网络模式

## Reactor 与 Proactor

**Reactor模式**：IO复用结合线程池，收到事件后分发给事件处理线程。多个连接共用一个阻塞对象，当某个连接有新的数据可以处理时，操作系统通知应用程序，线程从阻塞状态返回，将连接完成后的业务处理任务分配给线程进行处理。

- 单Reactor单线程/进程：Reactor负责事件分发，连接建立事件交给Acceptor获取连接，然后分配给Handler
- 单Reactor多线程
- 多Reactor多线程：（主从Reactor）父线程只需要接收新连接，子线程完成连接监听、分发等

**Proactor模式**：异步网络模式，基于已完成的IO事件处理

网络IO：

- 同步指内核数据拷贝到用户缓冲区的过程，read和send都是同步的
- 同步阻塞BIO: 一个请求创建一个线程处理
- 同步非阻塞NIO(Reactor): read在调用后立即返回，但数据准备好后拷贝仍是同步的。面向缓冲区，一个线程可以处理多个操作，基于Channelyu与Buffer，Selector监听多个通道的事件
- 异步非阻塞AIO(Proactor)：内核数据准备与数据拷贝都不需要等待

## Netty

[Netty学习手册](https://dongzl.github.io/netty-handbook/#/_content/chapter05)

Netty基于主从Reactors做了改进，其中主从Reactor多线程模型有多个Reactor。BossGroup线程维护Selector，只关注Accecpt，接受客户端连接。接收到Accept事件封装成NIOScoketChannel并注册到Worker线程，Worker线程负责网络的读写，Worker线程监听事件并进行处理。

BossGroup 和 WorkerGroup 类型都是 NioEventLoopGroup，NioEventLoopGroup 包含多个 NioEventLoop，每个 NioEventLoop 是一个线程，用于监听绑定在其上的 socket 的网络通讯。

## RPC

将rpc方法写在接口内，通过动态代理创建实现接口的对象，每次调用invoke设置当前session，并加入到ConcurrentLinkedQueue中。rpc维护一个发送和接收队列的上下文。新开一个线程轮询队列，处理发送和接收请求。

连接时设置解码器LengthFieldBasedFrameDecoder，在字节流中提取出消息的长度字段，并按照长度字段的值来解码消息。设置消息接收和发送的处理器。前4个字节作为长度，再4个字节作为flag，设置是否压缩信息。

- Proxy.newProxyInstance()，运行期动态创建接口对象
- method.getDeclaringClass() 获取声明方法的类

序列化：使用netty的ByteBufOutputStream，将其传入ObjectOutputStream.writeObject(Object obj)将对象写入ByteBuf中

## 单例模式

线程安全的单例模式实现：

1. 双重同步锁

```java
public class Singleton {
 
    private static volatile Singleton singleton;
 
    private Singleton() {
    }
 
    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}

```

2. 内部静态类，可能存在反射攻击和反序列化攻击

```java
private Singleton() {
  
}

public static Singleton getInstance() {
    return SingletonFactory.instance;
}

private static class SingletonFactory {
    private static Singleton instance = new Singleton();
}
```

3. 枚举，不能延时加载，能防止反射和反序列化调用

```java
public enum Singleton {
    INSTANCE;

    public void doSomething() {
        System.out.println("doSomething");
    }
}
```
