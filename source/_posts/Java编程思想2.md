---
title: Java编程思想 - 控制执行流程
toc: true
comments: true
date: 2019-02-25 10:38:49
tags:
  - Java
  - 控制流
category:
  - 编程
  - 读书笔记
  - Java编程思想
---

Java使用了C的所有流程控制语句，所以本文主要对Java中与C语法不同的地方进行介绍。

<!-- more -->

## Foreach语法
对于任何Iterable对象，可以使用Foreach语法进行迭代
```java
import java.util.*

public class ForEachFloat {
    public static void main(String[] args) {
        Random rand = new Random(47);
        float[] f = new float[10];
        for(int i = 0; i < 10; i++) {
            f[i] = rand.nextFloat();
        }
        for(float x : f) {
            System.out.println(x);
        }
    }
}
```

## goto
Java中goto依旧是一个关键字，但是并不起任何作用，Java中使用标签有了很大的限制，标签起作用的唯一地方刚好是在迭代语句之前，使用break和continue后跟标签的形式进行跳转，规则如下：
* 一般continue会退回最内层循环的开头，并继续执行
* 带标签的continue会到达标签的位置，并重新进入紧接在那个标签后面的循环，若紧接在后面的循环为for循环，则有递增。
* 一般的break会中断并跳出当前循环。
* 带标签的break会中断并跳出标签所指的循环。

```java
/**
 * LabeledFor
 */
public class LabeledFor {
    public static void main(String[] args) {
        int i = 0;
        outer:
        for(; true; ) {
            inner:
            for(; i < 10 ; i++) {
                System.out.println("i = " + i);
                if(i == 2) {
                    System.out.println("continue"); 
                    continue;
                }
                if(i == 3) {
                    System.out.println("break"); 
                    i++;
                    break;
                }
                if(i == 7) {
                    System.out.println("continue outer");
                    i++;
                    continue outer;
                }
                if(i == 8) {
                    System.out.println("break outer"); 
                    break outer;
                }
                for (int k = 0; k < 5; k++) {
                    if (k == 3) {
                        System.out.println("continue inner");
                        continue inner; 
                    }
                }
            }
        }
    }
}
```

## switch
* 从Java7开始，switch开始支持String，字符串的switch 是通过equals()和hashCode()方法来实现的，首先使用hashCode进行switch，然后通过使用equals()方法进行安全检查。
* Java5中提供的枚举类型enum可以和switch协同工作