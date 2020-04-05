> 《深入理解Java虚拟机：JVM高阶特性与最佳实践》周志明

### 1. Java内存管理

#### 1.1 运行时数据区域
> 《Java虚拟机规范》

![Java运行时数据区](https://images2015.cnblogs.com/blog/1182497/201706/1182497-20170616192740650-1039521219.png)

- **程序计数器**

PCR是一块较小的内存空间，起到指示当前进程所执行字节码行号的作用。对于OS来讲，任意时刻处理器的一个核心只会执行一条线程中的指令，即每条线程都需要一个独立的程序计数器独立存储，故称作“线程私有”内存。

- **虚拟机栈**

VM Stack描述的是Java方法执行的内存模型：每个方法执行时都会创建一个栈帧（Stack Frame）用于存储局部变量表、操作栈、动态链接和方法出口等信息。

局部变量表存在编译器可知的基本类型、对象引用和returnAddress（指向一条字节码指令地址）类型，故局部变量表所需内存空间在编译期完成分配也完成确定，方法运行期间不变。

- **本地方法栈**

类似于虚拟机栈，区别在于本地方法栈为虚拟机使用到的Native方法服务。

- **Java堆**

Java Heap是被所有线程共享的一块区域，在VM启动时创建，用于存放对象实例。根据JVM规范，其分配的内存无需物理连续。

"The heap is the runtime data area from which memory for all class instances and arrays is allocated."

- **方法区**

方法区用于存放已被VM加载的类信息、常量、静态变量和即时编译后的代码等数据。虽然规范中将方法区描述为堆的一个逻辑部分，但其有个别名NonHeap（非堆）用于区分Java堆的目的。

- **运行时常量池**

Runtime Constant Pool是方法区的一部分，用于存放Class文件中编译期生成的各种字面量和符号引用。同时也支持在运行期将新常量放置池中（String）。

- **直接内存**

Direct Memory并非虚拟机运行时数据区的一部分，但被频繁使用。

#### 1.2 对象访问

1. 通过句柄访问对象
2. 直接指针访问对象

#### 1.3 OutOfMemory
> 根据规范描述，除了程序计数器，其它运行区域都有OOM的可能。

### 2. 垃圾回收器
> GC 并非伴随Java产生，1960年诞生的Lisp是第一门真正应用内存动态分配和垃圾收集技术的语言。

#### 2.1 对象死亡
> 对象死亡：即不可能在被任何途径使用的对象。

- **引用计数法**

Reference Counting 实现简单，判断效率高，但很难解决对象间循环引用的问题，故Java并未选用。

- **跟搜索法**

通过一系列根对象作为起点，通过引用链向下搜索，但一个对象不可达时，证明该对象不可用，可判断为回收对象。

在Java中可做根对象的类型：虚拟机栈（本地变量表）中引入对象、方法区中类静态属性引入对象、方法区中常量引入对象以及本地方法栈中JNI引入对象。

#### 2.2 对象引用

- **强引用（Strong Reference）**：普遍存在的对象。GC永远不会回收被强引用的对象。
- **软引用（Soft Reference）**：用于描述还有用，但非必须的对象。系统在OOM之前会对软引用对象进行第二次回收。
- **弱引用（Weak Reference）**：被弱引用关联的对象只能生存到下一次GC之前。
- **虚引用（Phantom Reference）**：虚引用又称幽灵引用或者幻影引用，完全不影响关联对象的生存周期。唯一目的是对象被GC回收是会收到一个系统通知。

生存还是毁灭？

如果对象覆写如果对象覆写finalize方法，会被放置在一个名为F-Queue的队列中等待执行。VM会对队列中对象进行二次小规模筛选，故finalize()是逃离毁灭的最后机会。

#### 2.3 垃圾收集算法

- **标记-清除算法**

Mark-Sweep 是最简单的收集算法，但效率不高，且在清除后产生大量内存碎片。

- **复制算法**

将可用内存空间均分两份，当一份空间耗尽则将内部存活对象赋值到另一块区域，然后清除当前内存空间。此算法将内存缩小一半，代价实属太高。

现代商业虚拟机都是采用此算法回收新生代。但并非对等划分空间，而是分为一块较大的Eden和两块较小的Survivor空间，每次使用Eden和其中一块Survivor空间。

- **标志-整理算法**

根据老年代对象存活率高的特点，在收集过程中将存活对象向一端整理移动，最后清理掉边界以外内存。

- **分代收集算法**

当前大多商业虚拟机采用的算法，根据对象存活周期划分不同内存空间，再根据不同空间特点采用合适的收集回收算法。

#### 2.4 垃圾收集器
> 垃圾收集器是内存回收的具体实现。

![HotSpot虚拟机垃圾收集器](https://img2018.cnblogs.com/blog/1326194/201810/1326194-20181017145352803-1499680295.png)
（注：连线的两个收集器之间可以搭配使用）

- **Serial**

Serial收集器是最基本的、发展历史最悠久的收集器。特点是单线程、简单高效，但意味着收集器进行垃圾回收时，必须暂停其它所有的工作线程，直到回收结束（Stop The World）。

- **ParNew**

ParNew收集器其实就是Serial收集器的多线程版本。特点是默认开启与CPU数量相同的线程数。

![](https://img2018.cnblogs.com/blog/1326194/201810/1326194-20181017170542230-1929674942.png)

- **Parallel Scavenge**

与吞吐量关系密切，故也称为吞吐量优先收集器。属于新生代收集器也是采用复制算法的收集器，又是并行的多线程收集器。

- **Serial Old**

Serial Old是Serial收集器的老年代版本。同样是单线程收集器，采用标记-整理算法。

- **Parallel Old**

是Parallel Scavenge收集器的老年代版本。

- **CMS**

一种以获取最短回收停顿时间为目标的收集器。基于标记-清除算法实现，拥有并发收集、低停顿的特点。

![](https://img2018.cnblogs.com/blog/1326194/201810/1326194-20181017221500926-2071899824.png)

适用于注重服务的响应速度，希望系统停顿时间最短，给用户带来更好的体验等场景下，如web程序、b/s服务。

- **G1**

Garbage First收集器是一款面向服务端应用的垃圾收集器。

同时也是当前收集器技术发展最前沿的成果：
1. 并行与并发：G1能充分利用多CPU、多核环境下的硬件优势，使用多个CPU来缩短停顿时间。
2. 分代收集：G1能够独自管理整个Java堆，并且采用不同的方式去处理新创建的对象和存活的旧对象，以获取更好的收集效果。
3. 空间整合：G1运作期间不会产生空间碎片，收集后能提供规整的可用内存。
4. 可预测的停顿：G1除了追求低停顿外，还能建立可预测的停顿时间模型。能让使用者明确指定在一个长度为M毫秒的时间段内，消耗在垃圾收集上的时间不得超过N毫秒。

#### 2.5 分配和回收策略
 - 对象优先在Eden分配，当空间不足时发起一次Minor GC。
 - 大对象直接进入老年代：需要连续内存。
 - 长期存活对象进入老年代：默认15岁。
 - 动态对象年龄判定：当Survivor空间中相同年龄对象总超过空间大小的一半，则不低于该年龄的对象直接进入老年代。
 - 空间分配担保：发生Minor GC时VM会检测之前每次晋升到老年代的平均大小是否超过老年代剩余空间大小，超过则进行Full GC。如果未超过，则看是否允许担保（HandlePromotionFailure）配置，如果允许则只进行一次Minor GC。

### 3. 虚拟机监控
#### 3.1 JDK命令行工具

|名称|描述|说明|
|--|:--:|--:|
|jps|JVM Process Status Tool|显示指定系统内所有HotSpot VM进程|
|jstat|JVM Statistics Monitoring Tool|收集VM各方面数据|
|jinfo|Configuration Info for Java|显示虚拟机配置|
|jmap|Memory Map for Java|生成虚拟机内存转储快照（heapdump）|
|jhat|JVM Heap Dump Browser|分析heapdump文件|
|jstack|Stack for Java|显示虚拟机线程快照|

#### 3.2 JDK可视化工具

- **JConsle**（Java Monitoring and Management Console）：基于JMX的可视化监控和管理平台。
- **VisualVM**（All-in-One Java Troubleshooting Tool）：基于NetBeans平台的插件。

### 4. 类文件结构
#### 4.1 Class文件结构
Class文件是一组以8位字节为基础单位的二进制流，中间不添加任何分隔符，按顺序紧密排列。
若需要占用8位以上空间，则按照高位在前的方式分割成若干8位字节进行存储。



### 5. 类加载机制
虚拟机的类加载机制：虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型。

与编译时链接的语言不同，Java类型的加载和链接过程都是在程序运行期间完成，虽增加了类加载的开销，但是提供了高度的灵活性。
#### 5.1 生命周期
1. 加载 Loading
2. 验证 Verification
3. 准备 Preparation
4. 解析 Resolution
5. 初始化 Initialization
6. 使用 Using
7. 卸载 Unloading

#### 5.2 加载过程
> 类加载过程，即加载、验证、准备、解析和初始化这五个阶段的过程。

#### 5.3 类加载器
> 通过一个类的全限定名获取描述此类的二进制字节流。

对于任意一个类，都需要由加载它的类加载器同类本身一起确立在JVM中的唯一性，即不同加载器加载同一个class文件，这两个类必定不等。

**双亲委派模型**
从虚拟机角度只有两种类加载器：
- 启动类加载器：C++语言实现，是虚拟机的一部分
- 其它类加载器：Java语言实现，独立虚拟机，均继承java.lang.ClassLoader

![](https://pics6.baidu.com/feed/b8014a90f603738d649f9e0e75127c55f919ecd0.jpeg?token=350c225e7426f8cc62948a0dbfe8e911&s=59A01D723F0A774915E1754F0200E0F2)

**Parents Delegation Model**：当一个类加载器收到加载请求时直接将请求委派给父类加载器执行，每个层次均是如此。当父类加载器反馈无法完成加载请求（搜索范围内未发现目标类）时，子加载类再尝试自己加载。

待补充：Tomcat四种类加载器...

### 6. 执行引擎
> JVM最核心的组成部分之一。

#### 6.1 运行时栈帧
> 用于支持虚拟机进行方法调用与执行的数据结构。

![](https://img-blog.csdnimg.cn/20181202232229397.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d5MTMxNDU2MDE=,size_16,color_FFFFFF,t_70)

- 局部变量表

局部变量表是一组变量值空间，用于存放方法参数和方法内部的局部变量。

- 操作数栈

操作数栈是一个后入先出（LIFO）栈，同局部变量表一样，其最大深度在编译期被写入Code属性max_stacks字段中。

大多虚拟机都令两个相邻栈帧出现一部分重叠，让下面栈帧的操作数栈部分数据与上面栈帧局部变量表部分数据重叠，此时方法调用可以共享一部分数据而无需额外参数复制传递。

- 动态链接

每个栈帧包含一个指向运行时常量池中该帧所属方法的引用，以支持在每次运行期间将方法的符号引用转换成直接引用（动态链接）。

- 方法返回地址

方法退出等同于当前栈帧出栈，恢复上层方法的局部变量表和操作数栈，将返回值压人调用者栈帧的操作数栈中，并调整PC计数器的值以指向方法调用之后的指令。

#### 6.2 方法调用
> 方法调用并不等同于方法执行，调用阶段唯一任务就是确定被调用方法的版本，暂不涉及方法内部的执行。

所有方法调用的目标方法在Class文件中都是常量池中的符号引用，只有符合“编译期可知，运行期不可变”的方法在类加载的解析阶段将其符号引用转换成直接引用，同时这类方法的调用被称为**解析**（Resolution）。

在Java语言中符合这一要求的只有静态方法和私有发放两大类。

##### 6.2.1 静态分派

重载（Overload）时虚拟机通过参数变量的静态类型而非实际类型作为判断依据，则VM在编译期就通过静态类型确定了那个重载类型，并写入字节码中。

- **虚方法**：编译期确认唯一调用版本的方法（静态方法、私有方法、构造器方法和父类方法四类）
- **非虚方法**：除final方法外的剩余所有方法

但往往确定的版本不是“唯一的”，而是只能确定一个“更合适的”版本。
``` java
public class Overload{
    public static void say(Object arg){
        System.out.println("hello object");
    }

    public static void say(int arg){
        System.out.println("hello int");
    }

    public static void say(char arg){
        System.out.println("hello char");
    }

    public static void main(String[] args){
        say('a');
    }
}
// -----output-----
// hello char
```
如果注释掉say(char arg)方法，则输出结果会变成`hello int`，此时发生一次自动类型转换。

##### 6.2.2 动态分派

重写（Override）是动态分派的一个常见实际应用。

- 单分派和多分派

方法的接收者和方法的参数被称为方法的**宗量**，在选择目标方法时依据的宗量数量可以划分为单分派和多分派。

``` java
public class Dispatch{
    static class A{};
    static class B{};

    public static class Father{
        public void select(A arg){
            System.out.println("father select a");
        }
        public void select(B arg){
            System.out.println("father select b");
        }
    }

    public static class Son extends Father{
        public void select(A arg){
            System.out.println("son select a");
        }
        public void select(B arg){
            System.out.println("son select b");
        }
    }

    public static void main(String[] args){
        Father father=new Fatther();
        Father son=new Son();
        father.select(new A());
        son.select(new B());
    }
}

// father select a
// son select b
```

- Java的静态分派属于多分派

在编译期，编译期根据静态类型和方法参数类型对目标方法进行选择，产生两条指向Father.select(A)和Father.select(B)的指令。

- Java的动态分配属于单分派

而在运行阶段，当执行到代码对应invokevirtual指令时，由于编译期已经决定目标方法签名必须为select(B)，此时传递参数已经不会对方法选择产生影响，唯一宗量是方法接收者的实际类型。

- 动态分派的实现

由于动态分派是非常频繁的动作，实际工作中基于性能考虑进行一定优化：为类在方法区中建立一个虚方法表（Virtual Method Table）和接口方法表（Interface Method Table）。

#### 6.3 基于栈的解释执行
> 虚拟机执行Java代码时都有解释执行（通过解释器执行）和编译执行（通过即时编译期产生本地代码执行）两种选择。

- 基于栈的指令集可移植性强，但相比基于寄存器的指令集执行速度较慢。
- 基于栈的解释器执行过程

### 7. 编译期优化
目前性能优化集中到后端的即时编译器中，这样可以对非javac产生的class文件起到同样的优化效果。
#### 7.1 Javac编译器
> javac本身就是一个由Java语言编写的程序。

#### 7.2 语法糖

##### 7.2.1 泛型与类型擦除
Java语言的泛型只存在源码中：在编译后的字节码中已被替换成原来的原生类型，并在相应地方插入强制类型转换，即该方法称之为类型擦除。

``` java
// 类型擦除前
Map<String,String> map=new HashMap<>();
map.put("hello","你好");

System.out.println(map.get("hello"));
```
通过字节码反编译：
``` java
Map map=new HashMap();
map.put("hello","你好");

System.out.println((String) map.get("hello"));
```

当泛型遇到重载：
``` java
public class Test{
    public static void method(List<String> list){
        System.out.println("invoke method(List<String> list)");
    }
   
    public static void method(List<Integer> list){
        System.out.println("invoke method(List<Integer> list)");
    }
}
```
上述代码无法通过编译：方法参数的类型擦除后都变成了原生类型List<E>，导致两个方法的特征标签变得一模一样。
``` java
public class Test{
    public static String method(List<String> list){
        System.out.println("invoke method(List<String> list)");
        return "";
    }
   
    public static int method(List<Integer> list){
        System.out.println("invoke method(List<Integer> list)");
        return 0;
    }
}
```
添加两个不同类型返回值后 “重载” 成功：
- Java语言层面：Java方法的特征签名只包括了方法的名称、参数顺序以及参数类型
  Two methods have the same signature if they have the same name and argument types.
- Java虚拟层面：Java虚拟机中定义的是针对字节码的，方法的特征签名包括方法的返回值以及可能抛出的受检查异常（Checked Exceptions）。

注：在JDK7中，javac的行为和ejc的一致了，这段代码将不能编译通过。若要想通过编译，则只能执行声明的第一个方法。

##### 7.2.2 自动拆装箱和遍历
``` java
public static void main（String[] args）{
    List<Integer> list-Arrays.asList(1,2,3,4);
    int sum=0;
    for(int i: list){
        sum+=i;
        System.out.println(sum);
    }
}
```
反编译后：
``` java
public static void main（String[] args）{
    List<Integer> list-Arrays.asList(new Integer[]{
        // 自动装箱
        Integer.valueOf(1),
        Integer.valueOf(2),
        Integer.valueOf(3),
        Integer.valueOf(4)
    });
    int sum=0;
    for(Iterator localIterator=list.iterator; localIterator.hasNext();){
        // 自动拆箱
        int i=((Integer) localIterator.next()).intValue();
        sum+=i;
        System.out.println(sum);
    }
}
```

##### 7.2.3 条件编译
当使用条件为常量的条件判断语句时，生成的字节码只包含条件对应的语句。

即根据布尔值常量值的真假，编译器将把分支中不成立的代码块清除掉，但这样只能实现语句块（Block）级的条件编译，而无法涉及整个Java类结构。

### 8. 运行期优化
>  即时编译器并非虚拟机必需，Java虚拟机规范也未规定、限制和指导JIT编译器实现

在部分商用虚拟机中，Java最初是通过解释器（Interpreter）进行解释执行，当虚拟机发现某个方法或者代码块执行频繁，则认为是热点代码（Hot Spot Code）。

为了提高热点代码执行效率，运行时虚拟机将这些代码编译成与本地平台相关的机器码，并进行多层次优化，完成改任务的编译器称为**即时编译器**（Just In Time Compiler），简称JIT。

#### 8.1 HotSpot的JIT

##### 8.1.1 编译和解释
> 解释器和编译器二者各有优势：
> **解释执行**：每执行一句字节码就将字节码翻译成机器码执行，优点是启动效率快，同时节约内存空间；
> **编译执行**：预先把所需字节码编译成本地机器码一起执行，提高程序运行效率。

同时解释器可以作为编译器激进优化时的一个“逃生门”角色，当优化后的编译执行遇到陷阱时可以通过逆优化（Deoptimization）退回解释执行状态。

因此整个虚拟机框架中二者常常配合工作。

![https://my.oschina.net/u/3378039/blog/2221667](https://oscimg.oschina.net/oscnet/4f4a37b9a4ee4690f69167986875a6ed55d.jpg)

HotSpot虚拟机中内置了两个即时编译器，分别称为Client Complier和Server Complier，
它会根据自身版本与宿主机器的硬件性能自动选择运行模式，用户也可以使用`-client`或
`-server`参数去强制指定虚拟机运行在Client模式或Server模式。

**JVM相关参数**
|参数|描述|
|--|--|
|-XX:CICompilerCount=n| 指定JIT编译器用来编译方法的线程数量|
|-XX:CompileThreshold=n| 指定一个方法的调用次数，以使HotSpot和JIT 编译器能编译它|
|-Xcomp| 指定JVM在第一次使用时把所有的字节码编译成本地代码. (即CompileThreshold=1)|
|-Xbatch| 在前台编译方法，直到编译完成方法才能执行|
|-Xint| 仅仅使用解释模式，不激活JIT编译器 (即CompileThreshold=0)|

##### 8.1.2 编译对象与触发条件
- 多次调用的方法
- 多次执行的循环体

第一种情况理所当然是以整个对象作为编译对象，而后一种情况依旧以整个方法（而非循环体），且编译发生在方法执行过程，即方法栈帧还在栈上，方法就被替换了，故称作**栈上替换**（On Stack Replacement，简称OSR编译）。

判断一段代码是不是热点代码，是否需要触发即时编译的行为称作热点探测（Hot SpotDetection）：

- **基于采样的热点探测**（Sample Based Hot Spot Detection）：虚拟机周期性检查各个线程的方法栈，如发现某些方法经常出现在栈顶，即为热点方法。该方法简单、高效；
- **基于计数的热点探测**（Counter Based Hot Spot Detection）：虚拟机为每个方法建立计数器来统计方法执行次数，结果更加精确、严谨。

HotSpot虚拟机采用基于计数的热点探测方法，因此为每个方法准备了两种计数器：
- 方法调用计数器（Invocation Counter）：统计方法调用次数
- 回边计数器（Back Edge Counter）：统计循环体代码执行次数

![https://www.cnblogs.com/wzj4858/p/9109908.html](https://img-blog.csdn.net/20160812101630575?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

方法调用计数器存在**热度衰减**（Counter Decay）：当超过一定时间限度（半衰期）方法的调用次数仍然不足以让它提交给即时编译器编译，那这个方法的调用计数器就会被减少一半。

进行热度衰减的动作是在虚拟机进行垃圾回收时顺便进行的，可以使用参数`-XX:-UseCounterDecay`来关闭热度衰减，还可设置半衰周期。

回边计数器没有热度衰减。

##### 8.1.3 编译过程
虚拟机的编译过程默认都是在后台运行的。

> 用户可以通过参数`-XX:-BackgroundCompilation`来禁止后台编译：当达到JIT的编译条件，执行线程向虚拟机提交编译请求后将会一直等待，直到编译过程完成后再开始执行编译器输出的本地代码。

- Client Compiler 它是一个简单快速的三段式编译器，主要关注于局部性优化，放弃许多耗时较长的全局优化手段。

- Server Compiler 则是专门面向服务端配置且充分优化的高级编译器，执行所有经典优化动作，如无用代码清除、循环展开、循环表达式外提、消除公共子表达式、常量传播、基本块重排序等。

#### 8.2 编译优化技术
> Jdk1.3 之后Javac去除`-o`选项，不会产生任何字节码级别代码优化。
> 推荐阅读2006年版“紫龙书”———《编译原理》

##### 8.2.1 公共子表达式消除
> Subexpression Elimination

如果一个表达式E经过计算，且结果到现在E中所有变量未发生变化，则E这次出现就成为了公共子表达式。

``` java
int d= (c*b)*12 + a + (a + b*c);

// c*b 与 b*c 是同一表达式，且计算期间b与c值不变
int d= E*12 + a + (a + E);

// 同时编译器还可以进行代数化简（Alegebraic Simplification)
int d= E*13 + a*2;
```

##### 8.2.2 数组边界检查清除
> Array Bounds Checking Elimination

尽可能将运行期检查移至编译器

##### 8.2.3 方法内联
> Method inlining

运行时将调用次数达到一定阈值的方法调用替换为方法体本身，从而消除调用成本，并为接下来进一步的代码性能优化提供基础。

换句话说，方法内联就是把被调用方函数代码”复制”到调用方函数中，来减少因函数调用开销的技术：

``` java
private int add4(int x1, int x2, int x3, int x4) { 
    return add2(x1, x2) + add2(x3, x4); 
} 

private int add2(int x1, int x2) { 
    return x1 + x2; 
}

// 运行一段时间后JVM会把add2方法去掉，并把代码翻译成：
private int add4(int x1, int x2, int x3, int x4) { 
    return x1 + x2 + x3 + x4;
}
```

##### 8.2.4 逃逸分析
> 逃逸分析（Escape Analysis）的基本行为就是分析对象的动态作用域

一个对象在方法中被定义后可能被外部方法引用（作为调用参数传递），称之为**方法逃逸**。

甚至可能被外部线程访问，称为**线程逃逸**。如果分析证明对象不会逃逸到方法或线程之外，则可能为该对象进行一些高效的优化。

|方法|描述|
|--|--|
|栈上分配（Stack Allocation）| 在栈上分配非逃逸对象，其内存空间随栈帧出栈而销毁，不用等待GC|
|同步消除（Synchronization Elimination）|非逃逸对象读写不存在竞争，可消除其同步措施|
|标量替换（Scalar Replacement）| 不可逃逸对象被拆解成若干被方法使用的成员变量代替|

### 9. Java内存模型
> 内存模型可以理解为在特定的操作协议下，对特定的内存和高速缓存进行读写访问的抽象。

#### 9.1 硬件的效率
物理机引入高速缓存以解决处理器与内存速度的矛盾，但也引入一个新的**缓存一致性**（Cache Coherence）问题。
![https://blog.csdn.net/liao1990/article/details/81176681](https://img-blog.csdn.net/20170312144847642?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveTg3NDk2MTUyNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
除此之外，为了使得处理器内部的运算单元能竟可能被充分利用，处理器可能会对输入代码进行**乱序执行**（Out-Of-Order Execution）优化，处理器会在计算之后将对乱序执行的代码进行结果重组，保证结果准确性。与处理器的乱序执行优化类似，Java虚拟机的即时编译器中也有类似的**指令重排序**（Instruction Recorder）优化。

#### 9.2 JMM
![https://blog.csdn.net/liao1990/article/details/81176681](https://img-blog.csdn.net/20170312145317899?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveTg3NDk2MTUyNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**Java内存模型中规定了所有的变量都存储在主内存中**，每条线程还有自己的工作内存（可以与前面将的处理器的高速缓存类比），线程的工作内存中保存了该线程使用到的变量到主内存副本拷贝，线程对变量的所有操作（读取、赋值）都必须在工作内存中进行，而不能直接读写主内存中的变量。

并且定义八个原子操作来进行主内存与工作内存间的具体交互：lock（锁定）、unlock（解锁）、read（读取）、load（载入）、use（使用）、assign（赋值）、store（存储）、write（写入）。

##### 9.2.1 特殊规则

- **volatile关键字**

**volatile只保证了Java内存的可见性，并不保证其一致性**：虽然变量每次都需要去主内存读取，但对共享变量的运算操作（读取后的下一步计算）并非原子操作，还是导致了非线程安全。

同时，还可以**使用volatile变量来禁止重排序优化**。

- **long和double的非原子性协议**

JMM允许虚拟机将未被volatile修饰的64位（Long型和Double型）数据的读写操作划分为两次32位操作进行，即虚拟机实现可以不保证64位数据类型的load、store、read和write四个操作的原子性。

好在目前商用虚拟机均选择对64位数据进行原子操作。

##### 9.2.3 多线程三特征

- **原子性（Atomicity）**：指一个原子操作在cpu中不可以暂停然后再调度，既不被中断操作，要不执行完成，要不就不执行。

原子性由Java内存模型来直接保证的原子性变量操作，大致可以认为基础数据类型的访问和读写是具备原子性的。

Java内存模型还提供了lock和unlock操作（并未直接开放给用户）来满足更大范围的原子性，提供了更高层次的字节码指令monitorenter和monitorexit来隐匿地使用这两个操作，反映到Java代码synchronized关键字，因此在synchronized块之间的操作也具备原子性。

- **可见性（Visibility）**：指当一个线程修改了共享变量的值，其它线程能够立即得知本次修改。

JMM的主内存和工作内存，解决了可见性问题（包括volatile、synchronize以及final）。

- **有序性（Ordering）**

Java内存模型中的程序天然有序性可以总结为一句话：如果在本线程内观察，所有操作都是有序的；如果在一个线程中观察另一个线程，所有操作都是无序的。前半句是指“线程内表现为串行语义”，后半句是指“指令重排序”现象和“工作内存主主内存同步延迟”现象。