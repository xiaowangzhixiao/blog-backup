---
title: Java编程思想 - 初始化杂谈
toc: true
comments: true
date: 2019-02-26 15:32:58
tags:
  - Java
  - 初始化
category:
  - 编程
  - 读书笔记
  - Java编程思想
---

本文对Java编程思想第五章初始化与清理中对类成员初始化的几种方式进行整理总结。
## 自动初始化
此部分初始化会在构造器执行前发生。
### 默认初始化
java类中的成员变量在类加载时会初始化为默认值，基本变量设为0，boolean设为false，对象引用设为null。
### 指定初始化
可以直接在定义类成员变量的地方为其赋值。
```java
public class InitialValues2 {
  boolean bool = true;
  char ch = 'x';
  byte b = 47;
  short s = 0xff;
  int i = 999;
  long lng = 1;
  float f = 3.14f;
  double d = 3.14159;

  Depth d = new Depth();

  //调用某方法来提供初值
  int i = f();
  int j = g(i); //其中i必须在前面定义过了

  int f() { return 11; }
  int g(int n) { return n * 10; }

} ///:~
```

## 构造器初始化
