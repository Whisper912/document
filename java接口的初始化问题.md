---
title: java接口的初始化问题
date: 2020-01-08 17:28:18
tags:
- 类加载机制
- JVM
categories:
- Java
---

在看《深入理解JVM虚拟机》一书时，类加载机制这一部分，书上说
```
“对类、接口的执行<clint>时并不需要执行父接口的<clint>方法，只有使用父接口定义的变量时，父接口才会初始化。接口的实现类初始化时也不会调用接口的<clint>”
```

我们知道，使用类的static final修饰的基本类型或String类型变量时，这种变量会在编译阶段被当做常量加入类常量池，运行时直接引用类常量池中的副本，如下的例子
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

最开始我的疑问是，interface中的变量修饰符都为public static final，那根据书上说的**只有使用父接口定义的变量时，父接口才会初始化**，那接口岂不是永远不会初始化了？
```java
interface TestInterface {
    int a = 2;
    Thread thread1 = new Thread(){
        {
            System.out.println("thread 1 init!");
        }
    };
    Thread thread2 = new Thread(){
        {
            System.out.println("thread 2 init!");
        }
    };
}

public class Main implements TestInterface  {

    public static void main(String[] args) {
        System.out.println(new Main().a);
    }
}
/** 运行结果如下
2
**/
```
这个例子与我的疑问相符合，使用变量`a`时，`thread1`和`thread2`并没有被初始化，也就是说`TestInterface`并没有被加载。

但是！上面的疑问是有错误的，只是被static final修饰的`基本类型`或`String类型`变量才会发生例子中出现的情况，我们可以测试其他类型的变量会不会发生这种情况呢，由于接口没有static代码块，我们可以用下面的代码来验证
```java
interface TestInterface {
    int a = 2;
    Thread thread1 = new Thread(){
        {
            System.out.println("thread 1 init!");
        }
    };
    Thread thread2 = new Thread(){
        {
            System.out.println("thread 2 init!");
        }
    };
}

public class Main implements TestInterface  {

    public static void main(String[] args) {
        System.out.println(new Main().thread1);
    }
}
/** 运行结果如下
thread 1 init!
thread 2 init!
Thread[Thread-0,5,main]
**/
```
可以看到`thread1`和`thread2`都被初始化了。

所以，结论就是，使用接口的基本类型或String类型变量时不会加载并初始化接口，使用非基本类型的变量会加载并初始化接口。
