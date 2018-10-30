---
layout:     post
title:      StringBuffer & StringBuilder
subtitle:   String in java 
date:       2018-10-29
author:     Alessio
header-img: img/PostBack_04.jpg
catalog: true
tags:
    - java
    - String
---
### StringBuffer

- 线程安全的可变字符序列。
- 一个类似于 String 的字符串缓冲区，但不能修改。
- 虽然在任意时间点上它都包含某种特定的字符序列，但通过某些方法调用可以改变该序列的长度和内容。
- 可将字符串缓冲区安全地用于多个线程。
- 可以在必要时对这些方法进行同步，因此任意特定实例上的所有操作就好像是以串行顺序发生的，该顺序与所涉及的每个线程进行的方法调用顺序一致。
- StringBuffer 上的主要操作是 `append` 和 `insert` 方法，可重载这些方法，以接受任意类型的数据

### StringBuilder

- 一个可变的字符序列。此类提供一个与 StringBuffer 兼容的 API，但不保证同步。
- 该类被设计用作 StringBuffer 的一个简易替换，用在字符串缓冲区被单个线程使用的时候（这种情况很普遍）。
- 如果可能，建议优先采用该类，因为在大多数实现中，它比 StringBuffer 要快。
- 在 StringBuilder 上的主要操作是 append 和 insert 方法，可重载这些方法，以接受任意类型的数据。

## 相关面试题 & 有趣的点

### JDK1.7 中有这样的一段注解 描述了 String 的拼接 “ + ” 号操作

```java

The Java language provides special support for the string           Java语言为字符串提供特殊支持

concatenation operator ( + ), and for conversion of                 连接运算符（+），以及转换

other objects to strings. String concatenation is implemented       其他对象到字符串。 字符串连接已实现

through the  StringBuilder (or  StringBuffer )                      通过StringBuilder（或StringBuffer）

class and its append method.                                        类及其 append 方法。

String conversions are implemented through the method               字符串转换通过 toString 方法实现

toString, defined by Object andinherited by all classes in Java.    toString 方法由 Object 定义，并由 Java 中的所有类继承。
```

例如：

```java
String a="a";  String b="b" ;
```

当执行

```java
String c=a+b
```

操作时  实际上是创建一个 `StringBuilder` 对象  再通过 `apend()` 进行拼接  最后调用 `toStirng()` 生成一个新的对象给 c

### String / StringBuffer / StringBuilder 的总结

- `String` 和 `StringBuffer` 、 `StringBuilder` 相比， `String` 是不可变的， `String` 的每次修改操作都是在内存中重新 `new` 一个对象出来，而 `StringBuffer` 、 `StringBuilder` 则不用，并且提供了一定的缓存功能，默认 16 个字节数组的大小，超过默认的数组长度时，则扩容为原来字节数组的长度*2+2。
- `StringBuffer` 和 `StringBuilder` 相比，`StringBuffer` 是 `synchronized` 的，是线程安全的，而 `StringBuilder` 是非线程安全的，单线程情况下性能更好一点；使用 `StringBuffer` 和 `StringBuilder` 时，可以适当考虑下初始化大小，较少扩容的次数，提高代码的高效性