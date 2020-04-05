> 《Java核心技术 卷Ⅰ》

### 1. 注解

Java注解又称Annotation。注解不同于注释，可以把注解理解为代码里的特殊标记，这些标记可以在编译，类加载，运行时被读取，并执行相应的处理。通过注解开发人员可以在不改变原有代码和逻辑的情况下在源代码中嵌入补充信息。

#### 1.1 注解作用

- 生成描述性文件，甚至新的类定义；
- 减轻编写模板代码的负担；
- 使代码更加干净易懂；
- 将由编译器来测试和验证的格式，存储有关程序的额外信息。如@Override，可以让编译器检查子类中是否有重写父类中的方法。

#### 1.2 注解分类

**1.2.1 内建注解**

- @Override：表示当前的方法定义将覆盖超类中的方法；
- @Deprecated：为编译器将发出警告，因为注解@Deprecated解释的是被弃用的代码；
- @SuppressWarnings：关闭不当编译器警告信息；
- @SafeVarargs：参数安全类型注解。它的目的是提醒开发者不要用参数做一些不安全的操作,它的存在会阻止编译器产生 unchecked 这样的警告。它是在 Java 1.7 的版本中加入的；
- @FunctionalInterface：**函数式接口**（就是一个具有一个方法的普通接口，如Runnable ）注解，这个是 Java 1.8 版本引入的新特性。

**1.2.2 元注解**

元注解是可以注解到注解上的注解，或者说元注解是一种基本注解，应用到其它的注解上。

**@Target**

表示该注解可以用于什么地方，可能的ElementType参数有：

- `CONSTRUCTOR`：构造器的声明
- `FIELD`：域声明（包括枚举实例）
- `LOCAL_VARIABLE`：局部变量声明
- `METHOD`：方法声明
- `PACKAGE`：包声明
- `PARAMETER`：参数声明
- `TYPE`：类、接口（包括注解类型）或枚举声明。

**@Retention**

当 @Retention 应用到一个注解上的时候，它解释说明了这个注解的的存活时间。

- `SOURCE`：注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视；
- `CLASS`：注解只被保留到编译进行的时候，它并不会被加载到 虚拟机中；（默认）
- `RUNTIME`：VM将在运行期间保留注解，因此可以通过反射机制读取注解的信息。

**@Document**

能够将注解中的元素包含到 Javadoc 中去。

**@Inherited**

并非说注解本身可以继承，而是说如果一个超类被 @Inherited 注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解。

**@Repeatable**

Repeatable 自然是可重复的意思。@Repeatable 是 Java 1.8 才加进来的，所以算是一个新的特性。

#### 1.3 定义注解

**1.3.1 注解的属性**

注解的属性也叫做成员变量。注解只有成员变量，没有方法。注解的成员变量在注解的定义中以“无形参的方法”形式来声明，其方法名定义了该成员变量的名字，其返回值定义了该成员变量的类型。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation{
   
    int id() default -1;
    String msg() defaul "hit";
}
//注解中拥有 id 和 msg 两个属性
//注解中属性可以有默认值，默认值需要用 default 关键值指定。
```

赋值的方式是在注解的括号内以 `value="" `形式，多个属性之前用 `,`隔开：

```java
@TestAnnotation(id=1,msg="hello annotation")
public class Test {
}
```

需要注意的是，使用@interface自定义注解时，自动继承了`java.lang.annotation.Annotation`接口，由编译程序自动完成其他细节。在定义注解时，不能继承其他的注解或接口。

方法的名称就是属性的名称，返回值类型就是参数的类型（返回值类型只能是8 种基本数据类型、Class、String、enum及它们的数组）。

**1.3.2 注解的提取**

注解通过反射获取。首先可以通过 Class 对象的 isAnnotationPresent() 方法判断它是否应用了某个注解

```java
public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) {}
```

然后通过`getAnnotation()` 方法来获取 Annotation 对象。

```java
public <A extends Annotation> A getAnnotation(Class<A> annotationClass) {}
//或者是 `getAnnotations()` 方法。
public Annotation[] getAnnotations() {}
```

**定义注解**

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface UserCase {

    String username() default "admin";
    String password() default "admin";
}
```

**工具类**

```java
public class UserSystem {

    // 1. 测试注解，抛出异常
    public static void register (User user) throws Exception{
        // 2. 获取用户类字节码文件
        Class clazz = user.getClass();
        // 3. 反射获取成员变量
        Field filedUsername=clazz.getDeclaredField("username");
        Field filedPassword=clazz.getDeclaredField("password");
        // 4. 反射获取注解
        UserCase userCase= (UserCase) clazz.getAnnotation(UserCase.class);
        if (null!=userCase){
            filedUsername.setAccessible(true);
            filedUsername.set(user,userCase.username());
            filedPassword.setAccessible(true);
            filedPassword.set(user,userCase.password());
        }else {
            throw new Exception("注解为空");
        }

    }
}
```

**使用注解**

```java
@UserCase(username = "enosh",password ="123456")
public class User {

    private String username;
    private String password;

    public void print(){
        System.out.println("username : "+username);
        System.out.println("password : "+password);
    }
}

//测试
public class TestUser {
    public static void main(String[] args) {
        User user=new User();
        try {
            UserSystem.register(user);
            user.print();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```



### 2. 反射

反射库（reflection library）提供了一个非常强大的工具集，能够编写动态编写Java代码的程序。

#### 2.1 Class类

在程序运行期间，Java运行时系统始终为所有的对象维护一个运行时的类型标记。保存这些信息的类被称作`Class`。

**获取class对象**

- Object类中的`getClass()`方法会返回一个Class类型的实例；
- 可以通过静态方法`Class.forName(“className”)`获取类名对应的Class对象；
- `T.class`将代表匹配的类对象，如`Class c=Double[].class`。

虚拟机为每个类型管理一个Class对象，可以来比较对象的类型：

```java
if(e.getClass()==Execption.class)

```

还可以使用newInstance()快速创建一个类的实例，调用默认构造器：

```java
String s="java.util.Date";
Object o=Class.forName(s).newInstance();


```

将forName()和newInstance()配合使用，可以根据存储在字符串中的类名创建一个对象。

#### 2.2 分析类结构

Java.lang.reflect包下含有三个类Filed、Method和Constructor分别用于描述类的域、方法以及构造器。

```java
Field[] getFields();
//返回包含Field对象的数组，这些对象记录了这个类或器超类的公有域
Field[] getDeclaredFields();
//返回记录这个类全部域的数组

Method[] getMethods();
//返回所有公有方法，包括父类的公有方法
Method[] getDeclaredMethods();
//返回此类或接口的全部方法，但不含继承自父类的方法

Constructor[] getConstructors();
//返回公有构造器数组
Constructor[] getDeclaredConstructors();
//返回对象类的所有构造器


```

三个类都有一个getName()方法，返回项目的名称。Field类有一个getType()方法返回所属域类型的Class对象。

三个类还有一个getModifiers()方法，返回一个整型数值，用于描述public和static这样的修饰符使用情况。

```java
Class getDeclaredClass();
//返回一个用于描述类的Class对象
Class[] getExceptionTypes();
//返回方法抛出异常类型的Class对象数组，用于Method和Constructor

Class[] getParameterTypes();
//返回描述参数类型的class对象数组，用于Method和Constructor
Class[] getReturnType();
//返回描述返回类型的Class对象，用于Method


```

#### 2.3 运行时分析对象

```java
void setAccessible(boolean flag);
//为反射对象设置可访问标志，flag为true时表示屏蔽Java语言的访问检查，使得对象私有属性可以被查询和设置
Boolean isAccessible();
//返回反射对象的可访问标志
static void setAccessible(AccessibleObject[] array, boolean flag);

Field getField(String name);
//返回指定名称的公有域
Field getDeclaredField(String name);
//返回类中声明的给的名称的域


```

调用任意方法：在Method类中有一个invoke方法，允许调用包装在当前Method对象中的方法。

```java
Object invoke(Object obj, Object... args);
//第一个参数是隐式参数，可以被忽略，其余对象提供显示参数


```



### 3. 异常

异常是Java提供的用于程序中处理错误的一种机制。

#### 3.1 异常分类

在Java设计语言中，所有异常对象都是派生于Throwable类的一个实例：

![](https://upload-images.jianshu.io/upload_images/5982616-4ab25f2cfc5ca7b8.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/600/format/webp)

- Error类层次结构描述了Java运行时系统内部错误和资源耗尽错误，程序不应抛出此类异常；
- Exception类层次分为两大分支：Runtime表示由程序错误导致的异常，另一类是而程序本身没问题，但由于像I/O此类错误导致的其他异常。（`RuntimeException`这个名字容易让人产生混淆，事实上目前所有的异常都发生在运行时）

此外，Java语言规范将派生于Error和RuntimeException的异常称作未检查（unchecked）异常，此类异常编译器不检查，也不捕获。剩余其它异常称作已检查（checked）异常，故需要通过编译器检查，即必须捕获并进行处理。

#### 3.2 异常处理

##### 3.2.1 抛出异常

 找到一个合适的异常类，创建这个异常类的对象并抛出：

```java
void testExecption() throws Exception(){
    ...
    //1. 找到一个合适的异常类
    //2. 创建这个异常类的对象并抛出
    throw new Exception();
    ...
}


```

##### 3.2.2 捕获异常

要想捕获一个异常，必须是由try/catch语句块。

```java
try{
    ...
}catch(Exception e | UnkonwHostsExecption un){
    //handle this execption
}catch(MyExecption my){
    //handle myexecption
}


```

如果try中抛出的异常在catch中没有声明，则这个方法会立即终止退出。

我们还可以在catch子句中抛出一个异常，目的是改变异常的类型。

```java
try{
    access the database
}catch(SQLExecption e){
    Throwable se=new MyExaecption("database error");
    se.initCause(e);
    throw se;
}


```

使用这样的包装技术，可以使用子系统中的高级异常，而不丢失原始异常数据。

##### 3.2.3 finally子句

在try子句抛出异常并退出方法之前，会执行finally子句，即我们可以在finally子句中关闭一些资源。因为无论是否产生异常或异常是否被捕获，finally子句中的代码都会被执行。

```java
try{
   ...
   return n*n;
   
}finally{
    return n;
}


```

值得注意的是，当finally子句中包含return语句时，会产生一种意想不到的结果。如上面一段代码所示，try子句中返回值是n的平方，但在方法真正返回前，会执行finally子句中的代码，返回n使其覆盖掉原来的返回值。

但finally子句也会遇到问题，比如清理资源的方法也会抛出异常。

Jdk1.7之后为这种结构提供了一个快捷方式，使用带资源的try语句，即资源实现AutoCloseable接口，待try块退出时，自动调用close()方法和关闭资源。

##### 3.2.4 堆栈跟踪

堆栈跟踪（stack trace）是一个方法调用过程的列表，包含程序执行过程中方法调用的特定位置。可以使用Throwable类的printStackTrace()方法访问堆栈跟踪的文本描述信息。

**throwable**

```java
Throwable(String message, Throwable cause);
//使用指定的“原因”构造一个Throwable对象
Throwable initCause(Throwable cause);
//将这个对象设置成“原因”，若该对象已经被设置成“原因”，则抛出异常，返回this引用

void addSuppressed(Throwable t);
//为此异常添加一个“抑制”异常，在待资源的try语句中，t是close方法抛出的异常


```



#### 3.3 使用技巧

- 异常处理不能代替简单的测试

```java
//简单测试
if(!s.isEmpty()) s.pop();
//异常退出
try{
    s.pop();
}catch(EmptyStackExecption e){
   
}


```

异常捕获花费实践远高于前者，因此只有在异常处理情况下使用异常机制。

- 不要过分细化异常
- 利用异常层次结构：不要只抛RuntimeExecption和捕Throwable异常
- 不要压制异常
- 早抛出，晚捕获



#### 3.4 使用断言

断言机制可以允许在测试阶段向代码插入一些检查语句，当代码发布时，这些插入的检查语句会被自动地移走。因此在一个具有自我保护能力的程序中，断言很重要。

**启动和禁用断言**

在默认情况下，断言是被禁用的。可以在程序运行时使用`-enableassertions`或`-ea`选项启动断言。

```shell
java -ea MyApp


```

启动和禁用断言是类加载器的功能，无须重新编译程序。当断言被禁时，Class Loader跳过断言，因此不会降低程序运行速度。



#### 3.5 使用日志

日志管理系统管理者一个名为`Logger.global`的默认日志记录器，可以使用System.out替换它，并通过调用info()方法记录日志信息。

```java
Logger.getGlobale().info("File->XXX");


```

**日志记录器级别**

- SEVERE
- WARNING
- INFO
- CONFIG
- FINE
- FINER
- FINEST

默认情况下只记录前三个级别，也可以通过`setLevel(Level.FINE)`设置其他级别，此时，FINE和更高级的日志都被记录下来。



### 4. 泛型

泛型程序设计（Generic programming）意味着编写的代码可以被很多不同类型的对象所重用。

#### 4.1 泛型使用

**泛型类**

泛型类引入一个类型变量T，并使用尖括号（<>）括起来，放在类名之后。泛型类可以有多个类型变量，如：

```java
public class Pair<T,U>{...}


```

**泛型方法**

还可以在普通类中定义一个带类型参数的简单方法，

```java
//定义泛型方法
public static <T> T getMiddle(T.. a){...}

//调用泛型方法
String middle = Test.<String>getMiddle("John","Jack","Tom");


```

大多数情况下类型参数<String>可以省略，编译器有足够的信息推到出所调用的方法。但偶尔仍会提示错误：

```java
String middle = Test.getMiddle("John","Jack","Tom");

double middle = Test.getMiddle(3.14, 9.0, 1);


```

编译器会自动打包参数为1个Double和2个Integer类型对象，然后寻找这些类的共同超类。

**限定类型**

有时需要对类或方法的类型变量加以限定。

```java
public static <T extend Comparable& Serializable> T min(T[] a);


```

使用`&`分隔限定类型，而类型变量使用逗号分隔。

#### 4.2 虚拟机中的泛型

Java虚拟机没有泛型类型对象——所有的对象都属于普通的类对象。

无论定义一个怎样的泛型，都会自动提供一个相应的原始类型（raw type）。原始类型的名字就是删去类型参数后的泛型类型名，擦除（erased）类型变量，并替换为限定类型（无限定类型的变量使用Object）。



#### 4.3 约束和局限性

- 不能用基本类型实例化类型参数

因此没有Pair<double>，只有Pair<Double>。原因是类型擦除，类型擦除后Pair类的域只有Object，而Object不能存储double值。

- 运行时类型查询只适用于原始类型

虚拟机中的对象总有一个特定的非泛型类型，所有的类型查询只产生原始类型。

```java
new Pair<String>().getClass()==new Pair<Integer>().getClass();
//true


```

- 不能创建参数化类型（泛型）的数组

```java
Pair<String>[] table=new Pair<String>[10];  //Error


```

这是因为擦除之后table类型是Pair[]，可以转换成Object[]：

```java
Object[] array=table;


```

数组会记住其元素的类型，如果试图存储其他类型元素，会抛出ArrayStoreExecption异常。

```java
array[0] = "Hello";


```

不过对于泛型，擦除的这种机制无效，可以以下赋值:

```java
array[0] = new Pair<String>();
//能够通过数组存储检查，但由于不允许创建参数化类型的数组，会抛出一个类型错误


```

可以声明通配类型的数组，然受进行类型转换：

```java
Pair<String>[] table=(Pair<String>[]) new Pair<?>[10];


```

- Varargs警告

由于Java不支持泛型类型的数组，但当向参数个数可变的方法传递一个泛型类型实例时：

```java
public static <T> void addAll(Collection<T> coll, T... ts);


```

为调用此方法，虚拟机必须建立一个Pair<String>类型的数组，虽然违反前面规则，但此情况下规则有所放松，只会抛出一个警告。

使用`@SuppressWarnings("unchecked")`或`@SafeVarargs`注解addAll方法。

- 不能实例化类型变量
- 泛型类的静态上下文中的类型变量无效

不能在静态域或方法中引用类型变量：

```java
public class Signleton<T>{
    pravite static T singleInstance; //ERROR
    public static T getInstance(){  //ERROR
        return singleInstance;
    }
}


```

- 不能抛出或捕获泛型类的异常
- 注意擦除后的冲突


