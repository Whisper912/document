---
title: Java8笔记
date: 2019-11-11 13:43:18
tags:
- Java8
categories:
- Java
---

### lambda表达式与函数式接口
它允许我们将函数体作为参数传给某个方法，最简单的lambda表达式由`逗号隔开的参数列表` `->符号` `语句块`组成，例如
```java
Arrays.asList( "a", "b", "d" ).forEach( e -> System.out.println( e ) );
```

参数类型e是由编译器推理得出的，也可以显式的指定，lambda表达式由返回值，如果lambda表达式中语句块只有一行，可以不用`return`显式返回，例如
```java
Arrays.asList( "a", "b", "d" ).sort( ( e1, e2 ) -> e1.compareTo( e2 ) );
```

函数接口指的是只有一个函数的接口，这样的接口可以隐式转换为Lambda表达式。java.lang.Runnable和java.util.concurrent.Callable是函数式接口的最佳例子。只要某个开发者在该接口中添加一个函数，则该接口就不再是函数式接口进而导致编译失败。为了克服这种代码层面的脆弱性，并显式说明某个接口是函数式接口，Java 8 提供了一个特殊的注解`@FunctionalInterface`（Java 库中的所有相关接口都已经带有这个注解了），举个简单的函数式接口的定义：
```java
@FunctionalInterface
public interface Functional {
    void method();
}
```
但是函数式接口中可以有default method和static mothod，default method只能由接口的实例调用，static method只能由`接口名.静态方法调用`


