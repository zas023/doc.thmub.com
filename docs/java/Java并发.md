### 1. 前言

#### 1.1 并发编程的优势

- 发挥多处理器的强大能力
- 建模的简单性
- 异步事件的简化处理
- 用户界面响应更加灵敏

#### 1.2 线程带来的风险

- **安全性问题**：在没有充分同步的情况下，多个线程中的操作执行顺序不可预测（UnsafeSequence）
- **活跃性问题**：当某个操作无法继续执行，便出现活跃性问题。如饥饿、死锁
- **性能问题**：活跃性意味着正确的事情终会发生，但效果并不够理想



### 2. 线程安全

> **安全性**：程序的行为于其规范完全一致
>
> 一个对象是否是线程安全的，却决于它能否被多个线程访问
>
> 无状态对象一定是线程安全的。非正式讲，对象的状态指存储在状态变量（实例或静态域）中的数据

#### 2.1 原子性

- **竞态条件**（Race Condition）：由于不恰当的执行时序而出现不正确的结果，即当某个计算的正确性取决于多个线程的交替时序时，便发生了竞态条件。
- 先检查后执行：Check-then-Act
- 和大多数并发错误一样，竞态条件并不总会发生错误，还需要不恰当的执行时序
- 若以原子方式执行操作，则可以避免竞态条件问题

#### 2.2 加锁机制

##### 2.2.1 内置锁

> 同步代码块（Synchronized Block)

```java
//object作为锁的对象引用，sychronized关键字可以修饰方法和代码块
sychronized (object){
    //保护的代码块
}
```

- 每一个Java对象都可以作为一个实现同步的锁，称作内置锁（Intrinsic Lock）或监视器锁（Monitor Lock）；
- 内置锁是互斥锁，即最多只有一个线程持有这种锁；
- 获取内置锁的唯一途径就是进入这个锁保护的代码块或方法中。

##### 2.2.2 重入

> 一个线程可以成功请求它已经持有的锁

- **重入**意味着获取锁的操作的粒度是” 线程 “，而非” 调用 “

- 重入的实现：每个锁都关联一个计数器和一个所有者线程

- 重入的作用：提升了加锁行为的封装性，简化并发代码

  ```java
  class Father{
 
      public synchronized void doSomething(){
          System.out.println("father's this is: "+this);
      }
  }
 
  class Son extends Father{
      @Override
      public synchronized void doSomething() {
          super.doSomething();
          System.out.println("son's this is: "+this);
      }
  }
  //----------------调用子类doSomething方法结果-------------
  father's this is: com.enosh.javastudy.concurrency.sync.Son@29453f44
  son's this is: com.enosh.javastudy.concurrency.sync.Son@29453f44
  ```

  1. 父类中的<this>的值和子类一致，是子类的引用
  2. 两次调用获取的锁都是子类的对象锁，即内置锁是可重入的

#### 2.3 用锁保护状态

> 锁可以保护代码路径以串行形式访问，可以通过锁来构造一些协议实现对共享状态的独占访问

- 每个共享的和可变的变量都应该只由一个锁来保护
- 对于每个包含多个变量的不变性条件，涉及的所有变量都需要由同一个锁来保护

#### 2.4 活跃性与性能

- 判断同步代码块的合理大小
- 耗时操作（IO）定不能持有锁



### 3. 对象的共享

#### 3.1 可见性

- **内存可见性**（Memory Visiblity）:我们不仅希望可以阻止另一个线程同时访问对象状态，而且希望当一个线程修改对象的状态后，其他线程能够发现状态的变化
- 在没有同步的情况下，编译器、处理器和运行时等都可能对操作的执行顺序进行一些意想不到的调整
- 加锁的含义不仅局限于互斥行为，还包括可见性

#### 3.2 Volatile

- 一种稍弱的同步机制，当将变量声明成Volatile类型后，虚拟机和运行时都会注意的这个变量时共享的，因此该变量上的操作不会与其他内存操作一起重排
- 加锁机制既可以确保可见性，又可以保证原子性，而volatile变量只能确保可见性
- 注意最该使用volatile的条件：
  1. 只有单个线程更新变量的值
  2. 该变量不会与其他变量一起纳入不变性条件
  3. 在访问变量时不需要加锁

#### 3.3 发布和逸出

> 1. **发布**（Publish）一个对象是指，使对象能够在当前作用域之外的代码中使用，如将对象引用传递到其它类的方法中
> 2. **逸出**（Escape）是指一个不应该发布的对象被发布

- 发布对象最简单的方法时将对象的引用保存到公有的静态变量中
- 发布一个对象时，该对象的非私有域中所有引用对象同样被间接发布
- 发布内部状态会破坏封装性和不变性条件，所以我们发布对象的同时注意控制对象的逸出
- 不要在构造函数中**启动**一个线程。因为this引用会被构造函数中创建的线程共享，而当前构造函数还没有完成，却可能运行了线程，导致this还没有构造完成却被运行，产生诡异问题

#### 3.4 线程封闭

> 当访问共享的可变数据时，通常需要同步。一种避免同步的方式就是不共享数据。如果仅在单线程内访问数据，就不需要同步，这种技术称为**线程封闭**<Thread  Confinement>

- **Ad-hoc 线程封闭**：维护线程封闭的职责完全由程序实现来承担，是非常脆弱的，因此在程序中尽量少使用，一般使用更强的线程封闭技术，比如栈封闭或者ThreadLocal类
- **栈封闭**：是线程封闭中的一种特例，在栈封闭中只能通过局部变量才能访问对象
- **ThreadLocal**：这个类可以将线程中的某个值与保存值的对象关联起来，通常用于防止可变的单例实例对象和全局变量进行共享

#### 3.5 不变性

> 满足同步需求的另一种方法是使用不可变对象（Immutable Object）

- 不可变对象一定是线程安全的
- 不可变性不等于将对象的所有域都声明为<final>类型，因为<final>类型的域中可以包含可变对象的引用

**Final域**：

- final域可以确保初始化过程的安全性，从而可以不受限制的访问不可变对象，并在共享是无须同步

#### 3.6 安全发布

- **线程安全容器**：Hashtable，sychronizedMap，sychronizedList，sychronizedSet，ConcurrentMap，Vector，CopyOnWriteArrayList，CopyOnWriteArraySet，BlockingQuene，ConcurrentLinkedQuene
- **事实不可变对象**：对象在技术上可变，但其状态在发布后不会再变化
- 安全共享对象：
  1. 不可变对象可以由任意机制发布
  2. 事实不可变对象需用安全方式发布，如使用安全的容器
  3. 可变对象必须通过安全方式发布，且必须是线程安全或使用某个锁保护



### 4. 对象组合

> 将现有的线程安全组件组合成更大规模的组件或程序

- **设计线程安全的类**：收集同步需求，依赖状态的操作，状态的所有权
- **实例封闭**：将数据封装在对象的内部
- **线程安全性的委托**：
- 在现有的线程安全类中添加功能
- 将同步策略文档化



### 5. 基础构建模块



### 6. 任务执行

#### 6.1 Executor框架

> 任务是一组逻辑工作单位，而线程则是使任务异步执行的机制

- 在Java中执行任务的主要抽象不是**Thread**，而是**Executor**，基于生产者——消费者模式
- 同时，当希望获得一种更加灵巧的执行策略时，使用Executor代替Thread

```java
public interface Executor{
    void excute (Runnable command);
}
```

**线程池**

| 类型                    | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| newFixedThreadPool      | 将创建一个固定大小的线程池，每当提交一个任务就创建一个线程   |
| newCachedThreadPool     | 创建一个可缓存的线程池，规模不受限制，当规模超过需求则回收空闲线程 |
| newSingleThreadExecutor | 单线程的Executor，若异常结束则创建新线程来代替               |
| newScheduledThreadPool  | 固定长度线程池，但可以延迟或定时执行任务                     |

**Executor生命周期**

- JVM只有在所以非守护线程结束后才会退出，所以无法正确关闭Executor会导致JVM无法结束
- **ExecutorService**拓展了Executor接口，添加了一些用于管理生命周期的方法

```java
public interface ExcutorService extends Executor{
    void shutdown();    //执行平缓的关闭过程，等待所有任务的执行
    List<Runnable> shutdownNow();    //执行粗暴的关闭过程，返回尚未启动的任务清单
    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
    //执行awaitTermination后会立即调用shutdown，同时产生关闭ExcutorService的效果
}
```

**Timer**

- 问题：
  1. Timer执行所有定时任务时只会创建一个线程，一个任务执行过长，将会破坏其他TimerTask的定时精确
  2. **线程泄露问题**：Timer线程不会捕获异常，当TimerTask抛出未检查的异常时，错误认为整个Timer都被取消，将终止定时任务
- 解决：
  1. 使用**ScheduledThreadPoolExecutor**代替
  2. 使用**DelayQuene**构建自己的调度服务

#### 6.2 Callable和Future

> Executor框架使用Runnable作为基本任务的表示形式，run可以写入日志或放入共享数据结构，但具有很大的局限性，并不能返回一个值或者抛出一个受检查的异常

- **Callable**是一个更好的任务抽象，为主入口点（Call）返回一个值或异常
- **Future**表示一个任务的生命周期，并提供相应的方法判断任务是否完成、获取结果、取消任务等

```java
public interface Callable<V> {
    V call () throws Exception;
}

public interface Future<V> {
    boolean cancel (boolean mayInterrupIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException, CancellationException;
    V get(long tiomout, TimeUnit unit) throws InterruptedException, ExecutionException, CancellationException;
   
    //get方法的行为取决于任务状态，若任务已经完成，立即返回或抛出异常，若任务尚未完成，get将会被阻塞直到任务完成
}
```

- ExecutorService中的所有submit方法都将返回一个Future，还可以为Runnable或Callable实例化一个**FutureTask**

#### 6.3 CompletionService

> CompletionService融合了Executor和BlockingQuene，将Callable任务提交给它，然后使用take和poll等操作获取已完成的结果，这些结果是封装好的Future

- ExecutorCompletionService实现CompletionService，并将计算部分委托给一个Executor

```java
private class QueneingFuture<V> extends FutureTask<V>{
    QueneingFuture(Callable<V> c){super(c);}
    QueneingFuture(Runnable<V> t,V r){super(t,r);}
   
    protected void done() {
        completionQuene.add(this);
    }
    //ExecutorCompletionService实现原理非常简单，在构造函数中创建一个BlockingQuene保存计算结果，提交任务时，首先将任务包装成一个FutureTask的子类QueneingFuture,再改写done方法
}
```

### 7. 取消和关闭

> Java没有提供任何机制来快速安全的终止线程，但它提供了**中断**<Interruption>这种协作机制，让一个线程终止另一个线程

#### 7.1 任务取消

- 取消操作的原因：用户请求、时间限制、应用事件、错误、关闭
- 一个可取消的任务必须拥有取消策略（Cancellation Policy）

```java
  //使用volatile类型的域保存取消状态
  public class Test implements Runable {
      private volatile boolean cancelled;

      public void run(){
          while (!cancelled){
              //To DO
              //调用阻塞方法后可能检查不到取消标志位，永远不会停止
          }
      }

      public void cancel(){cancelled = true;}
  }
```

##### 7.1.1 中断

> 线程中断是一种协作机制，线程可以通过此机制来通知另一个线程，告诉它在合适或者可能的情况下终止

```java
//Thread中的中断方法
public class Thread {
    //中断目标线程
    public void interrupt(){}
    //返回中断线程的状态
    public bollean isInterrupted(){}
    //清除当前线程的中断状态，并返回之前的值，这也是清除中断状态的唯一方法
    public static boolean interrupted(){}
}
```

- 调用interrupt并不意味着停止目标线程，而只是传递请求中断的信号，然后由线程在下一个合适的取消点中断自己
- wait，sleep，join等方法也将严格执行这种请求

#### 7.2 停止基于线程的服务

- 对于持有线程的服务，只要服务的存在时间大于创建线程方法的时间，就应该提供生命周期方法
- 关闭ExecutorService：shutdown（执行所有任务） 和shutdownNow（返回尚未启动的任务清单）
- “**毒丸**”对象：指放入队列的对象，当得到该对象时立即停止

#### 7.3 处理非正常线程终止

> 导致线程提前死亡的最主要的原因就是**RuntimeException**。
>
> 这种异常表示出现某种编程错误或其它不可修复的错误，通常不会被捕获，也不会在调用栈中逐级传递，而是默认输出到控制台，并终止线程

- **UncaughtExceptionHandler**：检查出某个线程由于未捕获的异常而终结情况

```java
public interface UncaughtExceptionHandler {
    void uncaughtException(Thread t, Throwable e);
}

public class XXXLogger implements Thread.UncaughtExceptionHandler {
    public void uncaughtException(Thread t, Throwable e){
        //to do something
    }
}
```

#### 7.4 JVM关闭

> - 正常关闭：最后一个非天使线程结束、调用System.exit()、平台特定方法
> - 强行关闭：调用Runtime.halt()、OS关闭JVM进程

**关闭钩子（Shutdown Hook）**

- 关闭钩子是指通过Runtime.addShutdownHook注册的但尚未开始的线程

在正常的关闭中，JVM首先调用所有已经注册的关闭钩子，但并不能保证关闭钩子的调用顺序。

关闭钩子可以用于实现服务或应用程序的清理，如删除临时文件、清理无法由OS自动清除的文件

```java
public void start() {
    Runtime.getRuntime().addShutdownHook(new Thread(){
        public void run() {
            // to do
        }
    });
}
```

**守护线程**

> **守护线程**：在JVM启动时所创建的所有线程中，除主线程以外其余都是守护线程（如垃圾回收器以及其他执行辅助工作的线程）
>
> **普通线程**：默认情况下，主线程创建的所有线程都是普通线程

- 二者区别仅存在于线程退出时发生的变化，当一个线程退出时，JVM会检查其他线程的状态，如果仅剩守护线程，JVM会进行正常退出；

- 当JVM停止时，所有存活的守护线程会被直接抛弃——既不执行finally代码块，也不执行回卷栈，JVM直接退出。

**终结器**

> 当不需要内存资源时，可以通过GC回收，但一些如文件句柄或套接字句柄的资源必须显式的交还给操作系统

- 垃圾回收期对定义了<finalize>方法的对象进行特殊处理：回收器释放它们后调用其finalize方法，从而保证一些持久化的资源被释放
- 避免使用终结器，因为终结器无法保证它们何时运行或是否运行。一般情况我们使用finally代码块和显式的close方法

### 8. 显示锁

> 并非取代内置锁，而是在内置锁机制不适用时作为可选择的高级功能

#### 8.1 ReentrantLock

```java
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException;
    void unlock();
    Condition newCondition();
}
//ReentrantLock实现Lock接口，提供域synchronized相同的互斥性和可见性
Lock lock = new ReentrantLock();
lock.lock();
try{
    //to do
} finally{
    lock.unlock();
}
```

- 轮询锁和定时锁
- 可中断的锁
- 非块结构的加锁：连锁式加锁（Hand-Over-Hand Locking）或锁耦合（Lock Coupling）

**公平性**

ReentrantLock的构造函数可以创建一个非公平的锁（默认）或一个公平的锁，而内置锁同样不会提供确定的公平性保证

**ReentrantLock和Synchronized**

在一些内置锁无法满足需求的情况下，如需要某些高级功能：可定时、可轮询、可中断、公平队列以及非块结构的锁时使用ReentrantLock，否则应该**优先使用Synchronized**

#### 8.2 读-写锁

- **ReentrantReadWriteLock**

```java
public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
}
```
