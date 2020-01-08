---
title: 深入理解Java虚拟机笔记(3)-JVM类加载机制
date: 2020-01-07 13:43:18
tags:
- 类加载机制
- JVM
categories:
- Java
---
以下内容是我自己对《深入理解JVM虚拟机》中第7章——虚拟机类加载机制的笔记📒。

### 类加载的时机
类从被加载入虚拟机内存到卸载出内存，生命周期包括：加载(Loading)、验证(Verification)、准备(Preparation)、解析(Resolution)、初始化(Initialization)、使用(Using)和卸载(Unloading)7个阶段。其中验证、准备、解析3个阶段统称连接(Linking)。
/-----------------------------------/
/                                   /
/       这里缺一张图                  /
/                                   /
/-----------------------------------/


这里要注意，类加载(Class Loading)和类的加载(Loading)不是一个概念，类加载包括加载(Loading)、验证、准备、解析、初始化五个部分，而JVM没有强制要求类的加载(Loading)的时机，但是规定了初始化的时机，所以面试中经常讨论的类加载时机一般是说类的初始化时机。
JVM规定有且只有以下5种方式，这些方式成为主动引用，其他都是被动引用。
1. 遇到new、getstatic、putstatic、invokestatic这四条指令码是，如果类没被初始化，触发初始化。（即使用new实例化一个对象，读取一个类的静态变量(final修饰的基本类型或String类型除外)，调用一个类的静态方法时</br>
（❗️final修饰的基本类型或String类型除外是因为，用final static修饰的基本类型或String类型变量在编译时期已经替换为对常量池的引用，与本来的类无关了）
2. 使用java.lang.reflect包中方法对类进行反射调用时，如果类没被初始化。
3. 初始化一个类时，如果父类还没被初始化，先初始化父类。
4. 启动虚拟机时，会先调用main()方法，虚拟机会先初始化包含main()方法的那个类。
5. 使用java.lang.invoke.MethodHandle的实例得到的类没有初始化，会初始化这个类。

#### 被动引用的例子🌰
🌰1
```Java
public class Super {
    static int value = 2;
    static {
        System.out.println("Super init");
    }
}

public class Sub extends Super {
    static {
        System.out.println("Sub init");
    }
}

public class Main {
    public static void main(String[] args) {
        System.out.println(Sub.value);
    }
}
/**   
只输出“Super init", 不输出”Sub init“, 因为对于静态字段，只有*直接*定义这个字段的类才会初始化
**/
```
🌰2
```Java
//使用上个例子的Super和Sub
public class Main {
    public static void main(String[] args) {
        Super[] superArray = new Sub[10];
    }
}
/**
啥都不输出，牛逼
**/
```
🌰3
```Java
public class FinalStaticVariableTest {
    static {
        System.out.println("FinalStaticVariableTest init");
    }
    public static final String HELLO = "Hello";
}

public class Main {
    public static void main(String[] args) {
        System.out.println(FinalStaticVariableTest.HELLO);
    }
}
/**
不输出"FinalStaticVariableTest init", 因为编译时期，就把HELLO的值“Hello”存入了Main这个类的类常量池，运行“FinalStaticVariableTest.HELLO”时，直接引用的常量池的值，与原来的类没有关系了。
**/
```

### 类加载的过程
#### 加载
加载阶段需要完成3个步骤
* 通过一个类的全限定名获取此类的二进制字节流
* 将字节流所代表的静态存储结构转化为方法区的运行时数据结构
* 在内存中生存一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口。
#### 验证
确保字节流中的信息符合当前虚拟机的要求，并且不会危害虚拟机自身安全
#### 准备
准备阶段会给在方法区给类的静态变量开辟空间，并附上零值。如果该静态变量的属性表中有ConstantValue属性，那该变量就直接初始化为ConstantValue的值。</br>
(如果一个变量同时被static final修饰，且类型为基本类型或String，这个变量的属性表中会生成ConstantValue属性)
#### 解析
解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。
#### 初始化
初始化阶段，JVM会根据程序给类变量初始化。（运行类的static块，给static变量赋值），这里插一嘴，\<clinit\>方法是虚拟机自动为类生成的类的构造方法，与之对应\<init\>是实例的构造方法，不是每个类都有\<clinit\>方法。
* 接口的初始化：接口没有static代码块，但接口中也可以对static变量赋值，所以也可以拥有\<clinit\>方法。但是注意，和类的初始化不同。执行接口的\<clinit\>方法不会执行父接口的\<clinit\>方法；只有使用父接口中的变量才会初始化父接口；接口的实现类初始化时也不初始化父接口。</br>
(接口的变量都是public static final的，在前面讲了static final修饰的基本类型和String类型的值会在类编译时被存入主调class的常亮池，且对这个变量的引用都会替换为对常量池的引用。所以使用接口的基本类型或String类型的变量也不会初始化接口，使用接口的非基本类型会初始化接口，为此我专门写了一篇验证的文章🔒🔒🔒🔒🔒🔒🔒🔒🔒)
* 虚拟机会保证一个类的\<clinit\>方法是线程安全的，也就是说多个线程同时初始化一个类时只有一个线程能够执行\<clinit\>方法，其他线程会被阻塞。

### 类加载器
#### 类与类加载器
在JVM中，一个类的全限定名和加载这个类的类加载器一起唯一确定一个类，就是说用不同的类加载器加载同一个类，在JVM中这两个类是不相等的(equals()、isInstance()返回false)。

从JVM的角度看，只有两种类加载器，一种是启动类加载器(Bootrap ClassLoader)，在HotSpot中，这个类加载器是用C++实现的，是虚拟机自身的一部分；另一种就是其他类加载器，这些类加载器有java语言实现，独立于虚拟机外部，全部继承自java.lang.ClassLoader.

从java开发者角度看，类加载器还可以细化，一般会用到以下三种系统提供的类加载器。
* 启动类加载器：和前面介绍的一样，负责加载\<JAVA_HOME>/lib中的类库。
* 扩展类加载器(Extension ClassLoader)：负责加载\<JAVA_HOME>/lib/ext中的类库
* 应用加载器(Appliacation ClassLoader)：加载用户类路径(ClassPath)下的类库，如果应用程序中没有自定义类加载器，一般就是使用该类加载器。

应用程序都是由这三种类配合加载的，还可以加入自定义的类加载器，它们的关系如下图。
/-----------------------------------/
/                                   /
/       这里缺一张图                  /
/                                   /
/-----------------------------------/

这种层次关系被称为双亲委派模型，双亲委派模型要求除了顶层的类加载器(Bootrap ClassLoader)), 每个类加载器都应当有自己的父类加载器。这里父子关系不是继承(Inheritance)是组合(Composition)来复用父加载器代码。

双亲委派模型的工作过程：如果一个类加载器收到了加载类的请求，先不直接去尝试加载这个类，而且是抛给上层的父类，每一层的类加载器都如此，只有父加载器无法完成加载请求(它的搜寻范围没有这个类)，子类加载器才会尝试加载类。