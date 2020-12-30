---
title: Spring表达式语言SpEL
date: 2018-08-21 19:58:28
tags:
- Spring
- SpEL
category:
- 编程
- Java
---

## 学习目标
- 知识
	- 熟悉Spring表达式语言的基本概念
	- 了解Spring表达式语言的主要用途
	- 清楚Spring表达式语言的应用场景
- 技能
	- 创建基于Java的表达式语言
	- 创建基于XML的表达式语言
	- 创建基于注释的表达式语言

<!-- more -->

## 入门介绍
### 基本概述
能在运行时构建复杂表达式、存取对象属性、对象方法调用等等
- 基本表达式
- 类相关表达式
- 集合相关表达式
- 其他表达式

### 示例分析
```java
ExpressionParser parser = new SpelExpresssionParser();
Expression expressio = parser.parseExpression("('Hello'+'word').concat(#end)");
EvaluationContext context = new StandardEvaluationContext();
context.setVariable("end","!");
System.out.println(expression.getValue(context));
```
### 工作原理
#### 概念
- 表达式
- 解析器
- 上下文
- 根对象及活动上下文对象

#### 工作原理
1. 定义表达式
2. 定义解析器ExpressionParser
	- 生成记号流
	- 生成抽象语法树
	- 生成Expression接口
3. 定义上下文对象
4. 根据表达式求值

#### 主要接口
- ExpressionParser接口
- EvaluationContext接口
- Expression接口

### 配置风格
#### XML风格的配置
定义Bean时注入，默认使用"#{SpEL表达式}"表示，其中"#root"根对象默认可以认为是ApplicationContext，只有ApplicationContext实现默认支持SpEL，获取对象属性其实是获取容器中的Bean。
#### 注解风格
使用@Value注解指定SpEL表达式

## 操作范围
### 字面值
最简单的SpEL表达式仅包含一个简单的字面值：
```xml
<property name="num" value="#{5}" />
```

### Bean及Bean的属性或方法
- 引用Bean本身
```xml
<porperty name="bean2" value="#{bean1}"/>
<porperty name="bean2" ref="bean1"/>
```
- 引用Bean的属性
```xml
<bean id="bean2" class="com.jike.***.Bean2">
	<property name="name" value="#{bean1.name}"/>
</bean>
```
同下：
```java
Bean2 bean2 = new Bean2();
bean2.setName(bean1.getName());
```
- 引用Bean的方法
```xml
<property name="name" value="#{bean2.getName()}"/>a
<property name="name" value="#{bean2.getName.toUpperCase()}"/>
<!-- 返回null时，不执行后面的方法 -->
<property name="name" value="#{bean2.getName?.toUpperCase()}"/>
```

### 类的方法和常量
```xml
<property name="pi" value="#{T(java.lang.Math).PI}"/>
<property name="randomNumber" value="#{T(java.lang.Math).random()}"/>
```

## 运算符
### 运算符类型
|运算符类型|运算符示例|
|:-:|:-:|
|数值运算|+、-、*、/、%、^|
|比较运算|<、>、==、>=、<=、lt、gt、eg、le、ge|
|逻辑运算|and、or、not、\||
|条件运算|?:(ternary)、?:(Elvis)|
|正则表达式|matches|

### 数值运算
加、减、乘、除、取余、乘方、字符串连接
```xml
<property name="" value="#{counter.total+42}"/>
<property name="" value="#{counter.total+42}"/>
<property name="" value="#{2*T(java.lang.Math).PI*counter.total}"/>
<property name="" value="#{counter.total/counter.count}"/>
<property name="" value="#{counter.total%counter.count}"/>
<property name="" value="#{T(java.lang.Math).PI*counter.total^2}"/>
<property name="" value="#{'张'+''+'三'}"/>
```

### 比较运算
```xml
<property name="equal" value="#{counter.total == 100}"/>
<property name="equal" value="#{counter.total eq 100}"/>
```

### 逻辑运算符
```xml
<property name="handsome" value="#{counter.total == 100 and counter.count gt 100}"/>
<property name="equal" value="#{counter.total eq 100}"/>
```

### 条件运算符
?:
```xml
<property name="handsome" value="#{person.height gt 170 ? true :false}"/>
<!-- 默认true时使用条件 -->
<property name="name" value="#{person.name ?: 'Tom'}"/>
```

### 正则表达式
matches运算符，匹配成功返回true
```xml
<property name="validEmail" value="#{admin.email matches '[a-zA-Z0-9]+@[z-aA-Z0-9.-]+\\.com'}"
```

## 集合操作
SpEL可以引用集合中的某个成员，具有基于属性值来过滤集合成员的能力：
- 访问集合成员
- 查询集合成员
- 投影集合

### 访问集合成员
使用uitl命名空间中的<util:list>元素定义一个List集合
```xml
<util:list id="cities">
	<bean class="" p:name="" p:state="IL" p:population="2853114"/>
	<bean class="" p:name="" p:state="1L" p:population="2853114"/>
	<bean class="" p:name="" p:state="1L" p:population="2853114"/>
	<bean class="" p:name="" p:state="1L" p:population="2853114"/>
	……
</util:list>
```

### 查询集合成员
```xml
	<!-- 查询人口大于100000的所有城市 -->
	<property name="BigCities" value="#{cities.?[population gt 100000]}"/> 
	<!-- 查询人口大于100000的第一个城市 -->
	<property name="BigCity1" value="#{cities.^[population gt 100000]}"/> 
	<!-- 查询人口大于100000的最后一个城市 -->
	<property name="BigCity2" value="#{cities.$[population gt 100000]}"/> 
```
### 投影集合
```xml
<property name="cityName1" value="#{cities.![name]}"/>
<property name="cityName1" value="#{cities.![name+','+state]}"/>
<property name="BigCitiesName" value="#{cities.?[population gt 100000].![name]}"/> 
```

