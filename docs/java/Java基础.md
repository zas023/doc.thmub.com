### 1. Java简介

Java是一门面向对象编程语言，不仅吸收了C++语言的各种优点，还摒弃了C++里难以理解的多继承、指针等概念，因此Java语言具有功能强大和简单易用两个特征。Java语言作为静态面向对象编程语言的代表，极好地实现了面向对象理论，允许程序员以优雅的思维方式进行复杂的编程。

#### 1.1 Java的三个体系

- **Java SE（Java Platform，Standard Edition）**

         Java SE 以前称为 J2SE。它允许开发和部署在桌面、服务器、嵌入式环境和实时环境中使用的 Java 应用程序。Java SE 包含了支持 Java Web 服务开发的类，并为 Java Platform，Enterprise Edition提供基础。

- **Java EE（Java Platform，Enterprise Edition）**

         Java EE 以前称为 J2EE。企业版本帮助开发和部署可移植、健壮、可伸缩且安全的服务器端 Java 应用程序。Java EE 是在 Java SE 的基础上构建的，它提供 Web 服务、组件模型、管理和通信 API，可以用来实现企业级的面向服务体系结构（service-oriented architecture，SOA）和 Web 2.0 应用程序。

- **Java ME（Java Platform，Micro Edition）**

         Java ME以前称为 J2ME。Java ME 为在移动设备和嵌入式设备（比如手机、PDA、电视机顶盒和打印机）上运行的应用程序提供一个健壮且灵活的环境。Java ME 包括灵活的用户界面、健壮的安全模型、许多内置的网络协议以及对可以动态下载的连网和离线应用程序的丰富支持。基于 Java ME 规范的应用程序只需编写一次，就可以用于许多设备，而且可以利用每个设备的本机功能。

#### 1.2 Java 的主要特性

简单、面向对象、分布式、健壮、安全、体系结构中立、可移植、解释型、高性能、多线程、动态。

### 2. Java基础语法

#### 2.1 基础语法

- **大小写敏感**：Java是大小写敏感的，这就意味着标识符Hello与hello是不同的。
- **类名**：对于所有的类来说，类名的首字母应该大写。如果类名由若干单词组成，那么每个单词的首字母应该大写，例如 MyFirstJavaClass 。
- **方法名**：所有的方法名都应该以小写字母开头。如果方法名含有若干单词，则后面的每个单词首字母大写。
- **源文件名**：源文件名必须和类名相同。当保存文件的时候，你应该使用类名作为文件名保存（切记Java是大小写敏感的），文件名的后缀为.java。（如果文件名和类名不相同则会导致编译错误）。
- **主方法入口**：所有的Java 程序由**public static void main(String args[])**方法开始执行。

#### 2.2 标识符

Java所有的组成部分都需要名字。类名、变量名以及方法名都被称为标识符。

- 所有的标识符都应该以字母（A-Z或者a-z），美元符（$），或者下划线（_）开始，如$salary、\_value
- 首字符之后可以是任何字符的组合
- 关键字不能用作标识符
- 标识符是大小写敏感的

#### 2.3 数组

数组是储存在堆上的对象，可以保存多个同类型变量，这些变量类型既可以存储基本数据类型，也可以存储引用数据类型。

**一维数组**

```java
int[] a；     定义了一个int类型的数组a；
int a[];      定义了一个int类型的a数组；
//推荐使用第一种定义方式。
```

**数组初始化**就是为数组中的数组元素分配内存空间，并为每个数组元素赋值。Java中的数组必须先初始化后使用。

```java
//动态初始化：只指定长度，由系统给出初始化值
int[] array = new int[4];

//静态初始化：给出初始化值，由系统决定长度
int[] array = new int[]{1, 2, 3, 4};
//可简写成
int[] array = {1, 2, 3, 4};
```

- **默认初始化：**数组是引用类型，它的元素相当于类的成员变量，因此数组分配空间后，每个元素也被按照成员变量的规则被隐士初始化。

**二维数组**

```java
int[][] arr = new int[3][2];
//定义了一个二维数组arr
//这个二维数组有3个一维数组，名称是arr[0],arr[1],arr[2]
//每个一维数组有2个元素，可以通过arr[m][n]来获取,表示获取第m+1个一维数组的第n+1个元素
```

#### 2.4枚举

Java 5.0引入了枚举，枚举限制变量只能是预先设定好的值。使用枚举可以减少代码中的bug。

### 3. Java数据类型

#### 3.1 基本数据类型

Java语言内置八种基本类型。六种数字类型（四个整数型，两个浮点型），一种字符类型，一种布尔类型。

| 类型    | 最小值       | 最大值            | 默认值 | 字节数 | 说明                             |
| ------- | ------------ | ----------------- | ------ | ------ | -------------------------------- |
| byte    | -128（-2^7） | 127（2^7-1）      | 0      | 1      | 有符号的，以二进制补码表示的整数 |
| short   | -2^15        | 2^15 - 1          | 0      | 2      | 有符号的以二进制补码表示的整数   |
| int     | -2^31        | 2^31 - 1          | 0      | 4      | 有符号的以二进制补码表示的整数   |
| long    | -2^63        | 2^63 -1           | 0L     | 8      | 有符号的以二进制补码表示的整数   |
| float   | 2^-149       | 2^128-1           | 0.0f   | 4      | 单精度、符合IEEE 754标准的浮点数 |
| double  | 2^1074       | 2^1024-1          | 0.0d   | 8      | 双精度、符合IEEE 754标准的浮点数 |
| boolean |              |                   | false  |        | 表示一位的信息                   |
| char    | '\u0000' (0) | '\uffff' (65,535) |        | 2      | 一个单一的16位Unicode字符        |

- 浮点数的默认类型为double类型，不能用来表示精确的值，如货币。但float在储存大型浮点数组的时候可节省内存空间。
- [Primitive Data Types](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)

#### 3.2 引用数据类型

Java为每种基本类型都提供了对应的封装类型，分别为：Byte、Short、Integer、Long、Float、Double、Character、Boolean。引用类型是一种对象类型,它的值是指向内存空间的引用，就是地址。

- 在Java中，引用类型的变量非常类似于C/C++的指针。引用类型指向一个对象，指向对象的变量是引用变量。这些变量在声明时被指定为一个特定的类型，变量一旦声明后，类型就不能被改变了。
- 基本数据类型的变量是存储在栈内存中，而引用类型变量存储在栈内存中，保存的是实际对象在堆内存中的地址，实际对象中保存这内容。
- 对象、String、数组都是引用数据类型。
- 所有引用类型的默认值都是null。
- 一个引用变量可以用来引用任何与之兼容的类型。

#### 3.3 Number类

在实际开发过程中，我们经常会遇到需要使用对象，而不是内置数据类型的情形。为了解决这个问题，Java语言为每一个内置数据类型提供了对应的包装类。所有的包装类（Integer、Long、Byte、Double、Float、Short）都是抽象类Number的子类。

![](https://7n.w3cschool.cn/attachments/image/20160401/1459506097805010.jpg)

**Math类**

Java 的 Math 包含了用于执行基本数学运算的属性和方法，如初等指数、对数、平方根和三角函数。

Math 的方法都被定义为 static 形式，通过 Math 类可以在主函数中直接调用。

#### 3.4 自动拆装箱

> Java 从 Jdk1.5 开始引入自动装箱和拆箱,使得基本数据类型与引用类型之间相互转换变得简单。

- **自动装箱**：Java自动将原始类型转化为引用类型的过程，自动装箱时编译器会调用valueOf()方法,将原始类型转化为对象类型。
- **自动拆箱**：Java自动将引用类型转化为原始类型的过程，自动拆箱时编译器会调用intValue(),doubleValue()这类的方法将对象转换成原始类型值。

**主要的发生情况**

一是赋值时:

```java
Integer a = 3; //自动装箱
int b = a; //自动拆箱
```

二是方法调用：

```java
public Integer query(Integer a){
   return a;
}
query(3); //自动装箱
int result = query(3); //自动拆箱
```

**带来的问题**

- **程序性能**

由于装箱会隐式地创建对象创建，因此千万不要在一个循环中进行自动装箱的操作，下面就是一个循环中进行自动装箱的例子，会额外创建多余的对象,增加GC的压力,影响程序的性能。

```java
Integer sum = 0;
 for(int i=0; i<1000; i++){
   sum+=i;
}
```

- **空指针异常**

```java
Object obj = null;
int i = (Integer)obj;
```

#### 3.5 对象比较(缓存池问题)

```java
Integer a = 120;
int b= 120;
Integer c = 120;
Integer d = new Integer(120);
System.out.println(a == b);   //true    t1
System.out.println(a == c);   //true    t2
System.out.println(a == d);   //false   t3

Integer e = 128;
Integer f = 128;
System.out.println(e == f);   //false   t4

//使用Java Decompiler工具反编译class文件：
Integer integer1 = Integer.valueOf(120);
byte b = 120;
Integer integer2 = Integer.valueOf(120);
Integer integer3 = new Integer(120);
System.out.println((integer1.intValue() == b));
System.out.println((integer1 == integer2));
System.out.println((integer1 == integer3));

Integer integer4 = Integer.valueOf(128);
Integer integer5 = Integer.valueOf(128);
System.out.println((integer4 == integer5));

```

1. t1产生的原因是编译器编译时会调用intValue()自动的将a进行了拆箱，结果肯定是true;
2. t3结果无论如何都不会相等的，因为new Integer(120)构造器会创建新的对象。
3. 对于t2和t4，查看jdk的源码：

```java
public static Integer valueOf(int i) {
    assert IntegerCache.high >= 127;
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

       发现在 Java 8 中，Integer 缓存池的大小默认为 -128\~127，对于-128~127之间的值会取缓存中的引用,通过缓存经常请求的值而显著提高空间和时间性能。 这就能解释t2结果返回true，而t4由于128不在缓存区间内，编译器调用valueOf方法会重新创建新的对象，两个不同的对象返回false。

**基本类型对应的缓冲池**

- boolean values true and false
- all byte values
- short  and int values between -128 and 127
- char in the range \u0000 to \u007F

#### 3.6 类型转换

整型、实型(常量)、字符型数据可以混合运算。运算中，不同类型的数据先转化为同一类型，然后进行运算。

**自动类型转换**

数字表示范围小的数据类型可以自动转换成范围大的数据类型。

```java
int i = 200;
long l = i;
```

从小类型到大类型可以自动完成。类型的大小关系如下图所示：

![](https://images2015.cnblogs.com/blog/892594/201602/892594-20160212163546964-613019656.jpg)

值得注意的是，自动转换也要小心数据溢出问题：

```java
int count = 100000000;
int price = 1999;
long totalPrice = count * price;

//编译没任何问题，但结果却输出的是负数，
//这是因为两个 int 相乘得到的结果是 int, 相乘的结果超出了 int 的代表范围。
//这种情况，一般把第一个数据转换成范围大的数据类型再和其他的数据进行运算。

int count = 100000000;
int price = 1999;
long totalPrice = (long) count * price;
```

**向下转换**时可以直接将 int 常量字面量赋值给 byte、short、char 等数据类型，而不需要强制转换，只要该常量值不超过该类型的表示范围都能自动转换。

**强制类型转换**

```java
short s = 199;
int i = s;// 199

double d = 10.24;
long ll = (long) d;// 10

//以上的转换结果都在我们的预期之内，属于正常的转换和丢失精度的情况，下面的例子就一样属于数据溢出的情况。

int ii = 300;
byte b = (byte)ii;
//300 已经超出了 byte 类型表示的范围，所以会转换成一个毫无意义的数字。
```

**类型提升**

所谓类型提升就是指在多种不同数据类型的表达式中，类型会自动向范围表示大的值的数据类型提升。

```java
long count = 100000000;
int price = 1999;
long totalPrice = price * count;

//price 为 int 型，count 为 long 型，运算结果为 long 型，运算结果正常，没有出现溢出的情况。
```

#### 3.7 参数传递

Java 的参数是以值传递的形式传入方法中，而不是引用传递。在将一个参数传入一个方法时，本质上是将对象的地址以值的方式传递到形参中。因此在方法中使指针引用其它对象，那么这两个指针此时指向的是完全不同的对象，在一方改变其所指向对象的内容时对另一方没有影响。

- 基本类型作为参数传递时，是传递值的拷贝，无论你怎么改变这个拷贝，原值是不会改变的。

  ```java
  public class Test {
         public static void main(String[] args) {
             int n = 3;
            
             System.out.println("Before change, n = " + n);    
             //Before change, n = 3
            
             changeData(n);
             System.out.println("After changeData(n), n = " + n);
             //After changeData(n), n = 3
         }
     
         public static void changeData(int nn) {
             n = 10;
         }
  }
  ```

- 对象作为参数传递时，是把对象在内存中的地址拷贝了一份传给了参数。

  ```java
  public class Test1 {
      public static void main(String[] args) {
          StringBuffer sb = new StringBuffer("Hello ");
          System.out.println("Before change, sb = " + sb);
          //Before change, sb = Hello
         
          changeData(sb);
          System.out.println("After changeData(n), sb = " + sb);
          //After changeData(n), sb = Hi World!
      }
      public static void changeData(StringBuffer strBuf) {
          strBuf.append("World!");
      }
  }
 
  public class Test2 {
      public static void main(String[] args) {
          StringBuffer sb = new StringBuffer("Hello ");
          System.out.println("Before change, sb = " + sb);
          //Before change, sb = Hello
         
          changeData(sb);
          System.out.println("After changeData(n), sb = " + sb);
          //After changeData(n), sb = Hello
      }
      public static void changeData(StringBuffer strBuf) {
          strBuf = new StringBuffer("Hi ");
          strBuf.append("World!");
      }
  }
  ```

  [StackOverflow: Is Java “pass-by-reference” or “pass-by-value”?](https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value)

### 4. Java变量类型

> 在Java语言中，所有的变量在使用前必须声明。声明变量的基本格式如下：

```java
//type identifier [ = value][, identifier [= value] ...] ;

int a, b, c;         // 声明三个int型整数：a、b、c。
int d = 3, e, f = 5; // 声明三个整数并赋予初值。
```

#### 4.1 局部变量

> 类的方法中的变量

- 局部变量声明在方法、构造方法或者语句块中；
- 局部变量在方法、构造方法、或者语句块被执行的时候创建，当它们执行完成后，变量将会被销毁；
- 访问修饰符不能用于局部变量；
- 局部变量只在声明它的方法、构造方法或者语句块中可见；
- 局部变量是在栈上分配的;
- 局部变量没有默认值，所以局部变量被声明后，必须经过初始化，才可以使用。

#### 4.2 实例变量

> 独立于方法之外的变量，不过没有 static 修饰。

- 实例变量声明在一个类中，但在方法、构造方法和语句块之外；
- 当一个对象被实例化之后，每个实例变量的值就跟着确定；
- 实例变量在对象创建的时候创建，在对象被销毁的时候销毁；
- 实例变量的值应该至少被一个方法、构造方法或者语句块引用，使得外部能够通过这些方式获取实例变量信息；
- 实例变量可以声明在使用前或者使用后；
- 访问修饰符可以修饰实例变量；
- 实例变量对于类中的方法、构造方法或者语句块是可见的。一般情况下应该把实例变量设为私有。通过使用访问修饰符可以使实例变量对子类可见；
- 实例变量具有默认值。数值型变量的默认值是0，布尔型变量的默认值是false，引用类型变量的默认值是null。变量的值可以在声明时指定，也可以在构造方法中指定；
- 实例变量可以直接通过变量名访问。但在静态方法以及其他类中，就应该使用完全限定名：ObejectReference.VariableName。

#### 4.3 类变量

> 亦称静态变量，独立于方法之外的变量，用 static 修饰。

- 类变量也称为静态变量，在类中以static关键字声明，但必须在方法、构造方法和语句块之外；
- 无论一个类创建了多少个对象，类只拥有类变量的一份拷贝；
- 静态变量除了被声明为常量外很少使用。常量是指声明为public/private，final和static类型的变量。常量初始化后不可改变；
- 静态变量储存在静态存储区。经常被声明为常量，很少单独使用static声明变量；
- 静态变量在程序开始时创建，在程序结束时销毁；
- 与实例变量具有相似的可见性。但为了对类的使用者可见，大多数静态变量声明为public类型；
- 默认值和实例变量相似。数值型变量默认值是0，布尔型默认值是false，引用类型默认值是null。变量的值可以在声明的时候指定，也可以在构造方法中指定。此外，静态变量还可以在静态语句块中初始化；
- 类变量被声明为public static final类型时，类变量名称必须使用大写字母。如果静态变量不是public和final类型，其命名方式与实例变量以及局部变量的命名方式一致；
- 静态变量可以通过：*ClassName.VariableName*的方式访问。



### 5. Java修饰符

像其他语言一样，Java可以使用修饰符来修饰类中方法和属性。主要有两类修饰符：

1. **访问控制修饰符 :** default, public , protected, private
2. **非访问控制修饰符 :** final, abstract, static，synchronized 和 volatile

#### 5.1 访问控制修饰符

**默认访问控制修饰符**

- 使用默认访问修饰符声明的变量和方法，对同一个包内的类是可见的。
- 接口里的变量都隐式声明为public static final,而接口里的方法默认情况下访问权限为public。

**私有访问控制修饰符**

- 私有访问修饰符是最严格的访问级别，所以被声明为private的方法、变量和构造方法只能被所属类访问，并且类和接口不能声明为private。
- 声明为私有访问类型的变量只能通过类中公共的getter方法被外部类访问。
- Private访问修饰符的使用主要用来隐藏类的实现细节和保护类的数据。

**公有访问修饰符：**

- 被声明为public的类、方法、构造方法和接口能够被任何其他类访问。
- 类所有的公有方法和变量都能被其子类继承。
- Java程序的main() 方法必须设置成公有的，否则，Java解释器将不能运行该类。

**受保护的访问修饰符**

- 被声明为protected的变量、方法和构造器能被同一个包中的任何其他类访问，也能被不同包中的子类访问。

#### 5.2 访问控制与继承

- 父类中声明为public的方法在子类中也必须为public。
- 父类中声明为protected的方法在子类中要么声明为protected，要么声明为public。不能声明为private。
- 父类中默认修饰符声明的方法，能够在子类中声明为private。
- 父类中声明为private的方法，不能够被继承。

#### 5.3 非访问修饰符

> 为了实现一些其他的功能，Java也提供了许多非访问修饰符。

**static修饰符**

static修饰符，用来创建类方法和类变量。

- **静态变量：**static关键字用来声明独立于对象的静态变量，无论一个类实例化多少对象，它的静态变量只有一份拷贝。静态变量也被称为类变量。局部变量不能被声明为static变量。
- **静态方法：**static关键字用来声明独立于对象的静态方法。静态方法不能使用类的非静态变量。静态方法从参数列表得到数据，然后计算这些数据。

**final修饰符**

用来修饰类、方法和变量，final修饰的类不能够被继承，修饰的方法不能被继承类重新定义，修饰的变量为常量，是不可修改的。

- **final变量：**
  1. final变量能被显式地初始化并且只能初始化一次。被声明为final的对象的引用不能指向不同的对象。
  2. 但是final对象里的数据可以被改变。也就是说final对象的引用不能改变，但是里面的值可以改变。
- **final方法：**
  1. 类中的Final方法可以被子类继承，但是不能被子类修改。
  2. 声明final方法的主要目的是防止该方法的内容被修改。
- **final类：**
  1. final类不能被继承，没有类能够继承final类的任何特性。

**abstract修饰符**

| 抽象类                                                       | 抽象方法                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 抽象类不能用来实例化对象，声明抽象类的唯一目的是为了将来对该类进行扩充。 | 抽象方法是一种没有任何实现的方法，该方法的的具体实现由子类提供。抽象方法不能被声明成final和static。 |
| 一个类不能同时被abstract和final修饰。如果一个类包含抽象方法，那么该类一定要声明为抽象类，否则将出现编译错误。 | 任何继承抽象类的子类必须实现父类的所有抽象方法，除非该子类也是抽象类。 |
| 抽象类可以包含抽象方法和非抽象方法。                         | 如果一个类包含若干个抽象方法，那么该类必须声明为抽象类。抽象类可以不包含抽象方法。 |

**synchronized修饰符**

该关键字声明的方法同一时间只能被一个线程访问。Synchronized修饰符可以应用于四个访问修饰符。

**volatile修饰符**

修饰的成员变量在每次被线程访问时，都强迫从共享内存中重读该成员变量的值。而且，当成员变量发生变化时，强迫线程将变化值回写到共享内存。这样在任何时刻，两个不同的线程总是看到某个成员变量的同一个值。一个volatile对象引用可能是null。

**transient修饰符**

序列化的对象包含被transient修饰的实例变量时，java虚拟机(JVM)跳过该特定的变量。

该修饰符包含在定义变量的语句中，用来预处理类和变量的数据类型:

```java
public transient int limit = 55;   // will not persist
public int b; // will persist

```



### 6. Java运算符

数学运算是计算机的最基本用途之一，作为一门计算机编程语言，Java也提供了一套丰富的运算符来操纵变量。

#### 6.1 算术运算符

| 加法 | 减法 | 乘法 | 除法 | 取模 | 自增 | 自减 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| +    | -    | *    | /    | %    | ++   | --   |

#### 6.2 关系运算符

| 相等 | 不等 | 大于 | 小于 | 大于等于 | 小于等于 |
| ---- | ---- | ---- | ---- | -------- | -------- |
| ==   | !=   | >    | <    | \>=      | \<=      |

#### 6.3 位运算符

> 变量A的值为60和变量B的值为13：

| 操作符 | 描述                                                         | 例子                           |
| ------ | ------------------------------------------------------------ | ------------------------------ |
| ＆     | 按位与操作符，当且仅当两个操作数的某一位都非0时候结果的该位才为1。 | （A＆B），得到12，即0000 1100  |
| \|     | 按位或操作符，只要两个操作数的某一位有一个非0时候结果的该位就为1。 | （A \| B）得到61，即 0011 1101 |
| ^      | 按位异或操作符，两个操作数的某一位不相同时候结果的该位就为1。 | （A ^ B）得到49，即 0011 0001  |
|      | 按位补运算符翻转操作数的每一位。                             | （A）得到-60，即1100 0011    |
| <<     | 按位左移运算符。左操作数按位左移右操作数指定的位数。         | A << 2得到240，即 1111 0000    |
| >>     | 按位右移运算符。左操作数按位右移右操作数指定的位数。         | A >> 2得到15即 1111            |
| >>>    | 按位右移补零操作符。左操作数的值按右操作数指定的位数右移，移动得到的空位以零填充。 | A>>>2得到15即0000 1111         |

#### 6.4 逻辑运算符

| 与   | 或   | 非   |
| ---- | ---- | ---- |
| &&   | \|\| | !    |

#### 6.5 赋值运算符

`=`是最简单的赋值运算符，将右操作数的值赋给左侧操作数，还可以与算数运算符和位运算符结合使用。

#### 6.6 条件运算符

条件运算符也被称为三元运算符。该运算符有3个操作数，并且需要判断布尔表达式的值。该运算符的主要是决定哪个值应该赋值给变量。

```java
variable x = (expression) ? value if true : value if false
```

#### 6.7 instanceOf 运算符

instanceOf 用于操作对象实例，检查该对象是否是一个特定类型（类类型或接口类型）。

```java
//( Object reference variable ) instanceOf  (class/interface type)

String name = 'James';
boolean result = name instanceOf String; // 由于name是Strine类型，所以返回真
```

如果被比较的对象兼容于右侧类型,该运算符仍然返回true。

```java
class Vehicle {}

public class Car extends Vehicle {
   public static void main(String args[]){
      Vehicle a = new Car();
      boolean result =  a instanceof Car;
      System.out.println( result);
   }
}
```

#### 6.8 运算符优先级

> 下表中的运算符优先级由上至下依次降低：

| 类别     | 操作符                                     | 关联性   |
| -------- | ------------------------------------------ | -------- |
| 后缀     | () [] . (点操作符)                         | 左到右   |
| 一元     | + + - ！                                 | 从右到左 |
| 乘性     | * /％                                      | 左到右   |
| 加性     | + -                                        | 左到右   |
| 移位     | >> >>>  <<                                 | 左到右   |
| 关系     | >> = << =                                  | 左到右   |
| 相等     | ==  !=                                     | 左到右   |
| 按位与   | ＆                                         | 左到右   |
| 按位异或 | ^                                          | 左到右   |
| 按位或   | \|                                         | 左到右   |
| 逻辑与   | &&                                         | 左到右   |
| 逻辑或   | \| \|                                      | 左到右   |
| 条件     | ？：                                       | 从右到左 |
| 赋值     | = + = - = * = / =％= >> = << =＆= ^ = \| = | 从右到左 |
| 逗号     | ，                                         | 左到右   |



### 7. Java String类

在 Java语言中有8种基本类型和一种比较特殊的类型`String`。由于`String`在Java世界中使用过于频繁，Java为了避免在一个系统中产生大量的String对象，引入了字符串常量池（当然基础类型也有）。

#### 7.1 常量池

常量池就类似一个Java系统级别提供的缓存，指的是在编译期被确定，并被保存在已编译的.class文件中的一些数据。它包括了关于类、方法、接口等中的常量，也包括字符串常量。Java会确保一个字符串常量只有一个拷贝。

```java
String s1 = "Programming";
String s2 = new String("Programming");
String s3 = "Program" + "ming";
String s4 = new String(s2);
String s5 = "Programming";

System.out.println(s1 == s2);//false
System.out.println(s4 == s2);//false
System.out.println(s1 == s3);//true
System.out.println(s1 == s5);//true
System.out.println(s1 == s1.intern());//true

s2.intern();
System.out.println(s1 == s2);//false 没有将返回值赋值给s2

s2=s2.intern();
System.out.println(s1 == s2);//true

```

- s1和s3中的”Programming”都是字符串常量，它们在编译期就被确定了，所以s1==s3为true；而”Program”和”ming”也都是字符串常量，当一个字符串由多个字符串常量连接而成时，JVM对此做了一个优化，s3也同样在编译期就被解析为一个字符串常量，所以s3也是常量池中 ”Programming”的一个引用。 所以我们得出s5==s1==s3
- 用new String() 创建的字符串不是常量，不能在编译期就确定，所以new String() 创建的字符串不放入常量池中，它们有自己的地址空间。所以有s2！=s4
- String对象的intern()方法会得到字符串对象在常量池中对应的版本的引用（如果常量池中有一个字符串与String对象的equals结果是true），如果常量池中没有对应的字符串，则该字符串将被添加到常量池中，然后返回常量池中字符串的引用。

**补充：**存在于.class文件中的常量池，在运行期被JVM装载，并且可以扩充。String的intern()方法就是扩充常量池的一个 方法；当一个String实例str调用intern()方法时，Java查找常量池中是否有相同Unicode的字符串常量，如果有，则返回其的引用， 如果没有，则在常量池中增加一个Unicode等于str的字符串并返回它的引用。

#### 7.2 String是不可变的

String 是一个典型的 Immutable 类，被声明成为 final class，所有属性也都是 final 的。也由于它的不可变，类似拼接、裁剪字符串等动作，都会产生新的 String 对象。

- 可以缓存Hash值：因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。
- 常量池的需要：如果一个String对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。
- 线程安全：String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

#### 7.3 equals() & ==

- String类已经重写过了equals方法，这对于String简单来说就是比较两字符串的Unicode序列是否相当，如果相等返回true。而==是比较两字符串的地址是否相同，也就是是否是同一个字符串的引用。
- 在符合数据类型中，则equals和==都是比较两对象的地址是否相同，除非重写equals，详见hashcode与equals的区别。

#### 7.4 Buffer&Builder

- StringBuffer和StringBuilder都实现了AbstractStringBuilder抽象类，拥有几乎一致对外提供的调用接口。
- 其底层在内存中的存储方式与String相同，都是以一个有序的字符序列（char类型的数组，JDK 9 以后是 byte）进行存储，不同点是StringBuffer/StringBuilder对象的值是可以改变的，并且值改变以后，对象引用不会发生改变。
- 两者对象在构造过程中，首先按照默认大小申请一个字符数组（这个大小是 16），由于会不断加入新数据，当超过默认大小后，会创建一个更大的数组，并将原先的数组内容通过arraycopy复制过来，再丢弃旧的数组。因此，对于较大对象的扩容会涉及大量的内存复制操作，如果能够预先评估大小，可提升性能。
- **在线程安全上，StringBuilder是线程不安全的，而StringBuffer是线程安全的。**区别仅在于最终的方法是否加了 synchronized，即三者的执行效率上 StringBuilder > StringBuffer > String 。

**小结：**

1. String适用于少量的字符串操作的情况。
2. StringBuilder适用于单线程下在字符缓冲区进行大量操作的情况。
3. StringBuffer适用多线程下在字符缓冲区进行大量操作的情况。



### 8. Object方法

#### 8.1 概况

```java
public native int hashCode()

public boolean equals(Object obj)

protected native Object clone() throws CloneNotSupportedException

public String toString()

public final native Class<?> getClass()

protected void finalize() throws Throwable {}

public final native void notify()

public final native void notifyAll()

public final native void wait(long timeout) throws InterruptedException

public final void wait(long timeout, int nanos) throws InterruptedException

public final void wait() throws InterruptedException

```

#### 8.2 equals()

**等价关系**

```java
//自反性
x.equals(x); // true

//对称性
x.equals(y) == y.equals(x); // true

//传递性
if (x.equals(y) && y.equals(z))
    x.equals(z); // true;

//一致性，多次调用 equals() 方法结果不变
x.equals(y) == x.equals(y); // true

//与 null 的比较
//对任何不是 null 的对象 x 调用 x.equals(null) 结果都为 false
x.equals(null); // false;

```

**等价与相等**

- 对于基本类型，== 判断两个值是否相等，基本类型没有 equals() 方法。
- 对于引用类型，== 判断两个变量是否引用同一个对象，而 equals() 判断引用的对象是否等价。

```java
public class EqualExample {

    private int x, y, z;
   
    @Override
    public boolean equals(Object o) {
        //检查是否为同一个对象的引用，如果是直接返回 true
        if (this == o) return true;
        //检查是否是同一个类型，如果不是，直接返回 false
        if (o == null || getClass() != o.getClass()) return false;

        //将 Object 对象进行转型
        EqualExample that = (EqualExample) o;

        //判断每个关键域是否相等
        if (x != that.x) return false;
        if (y != that.y) return false;
        return z == that.z;
    }
}

```

#### 8.3 hashCode()

- hashCode() 返回散列值，而 equals() 是用来判断两个对象是否等价。等价的两个对象散列值一定相同，但是散列值相同的两个对象不一定等价。
- 在覆盖 equals() 方法时应当总是覆盖 hashCode() 方法，保证等价的两个对象散列值也相等。

#### 8.4 toString()

默认返回 Example@4554617c 这种形式，其中 @ 后面的数值为散列码的无符号十六进制表示。

#### 8.5 clone()

**cloneable**

clone() 是 Object 的 protected 方法，它不是 public，一个类不显式去重写 clone()，其它类就不能直接去调用该类实例的 clone() 方法。

**浅拷贝**

拷贝对象和原始对象的引用类型引用同一个对象。

**深拷贝**

拷贝对象和原始对象的引用类型引用不同对象。

**clone() 的替代方案**

使用 clone() 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。Effective Java 书上讲到，最好不要去使用 clone()，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。