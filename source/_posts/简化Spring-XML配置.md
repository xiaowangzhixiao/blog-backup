---
title: 简化Spring XML配置
date: 2018-08-21 19:49:04
tags:
- Spring
category:
- 编程
- java
---

本节内容主要介绍了Bean的自动装配、基于注解的配置和基于Java类的配置对XML配置的简化。
<!-- more -->
## 自动装配Bean的属性
当Spring装配Bean的属性时，如果非常明确，则可以使用自动装配模式：
### 自动装配类型
手动：使用ref装配
策略：

- byName
	假设属性的名字和bean的名字相同
	```xml
	<bean id="customer" class="xxx" autowire="byName"/>
	<bean id="persion" class="xxx."/>
	```
- byType
	只允许存在一个bean符合类型相同
	```xml
	<bean id="customer" class="xxx" autowire="byType"/>
	<bean id="persion" class="xxx."/>
	```
	可以设置primary，默认true
	```xml
	<bean id="customer" class="xxx" autowire="byType"/>
	<bean id="persion" class="xxx." primary="false"/>
	```
	忽略某些候选类
	```xml
	<bean id="customer" class="xxx" autowire="byType"/>
	<bean id="persion" class="xxx." autowire-candidate="false"/>
	```
- constructor
	构造器注入,匹配构造器的入参类型：
	```xml
	<bean id="customer" class="xxx" autowire="constructor"/>
	<bean id="persion" class="xxx." />
	```
- autodetect
	最佳自动装配，首先尝试使用constructor，然后是byType
	```xml
	<bean id="customer" class="xxx" autowire="autodetect"/>
	<bean id="persion" class="xxx." />
	```

#### 默认自动装配
在根元素<beans>上配置default-autowire类型
```xml
	<beans default-autowire="byType">
```
#### 混合装配
可以显式装配和自动装配同时使用

## 基于注解的配置
### 注解配置示例
@Component是Spring容器中的基本注解，表示容器中的组件（bean），可以作用在任何层次：
```java
@Component("userDao")
public class UserDao{}
```
等效于
```xml
<bean id="userDao" class="xxx.UserDao"/>
```
##### 可用作定义Bean的注解
- @Component用于DAO实现类标注
- @Service用于service实现类进行标注
- @Controller用于controller实现类进行标注
- @Repository

### 加载注解配置
context命名空间提供了通过扫描类包来加载利用注解定义Bean的方式
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

	<context:component-scan bese-package="com.xxx.xxx"/>     
    ......

</beans>
```
过滤方式之resource-pattern
```xml
<context:component-scan bese-package="com.xxx.xxx" resoure-pattern="helloImpl/*.class"/> 
```
过滤方式之过滤子元素
```xml
<context:component-scan bese-package="com.xxx.xxx" resoure-pattern="helloImpl/*.class">
	<context:include-filter type="regex" expression="com\.xxx\.*"/>
	<context:exclude-filter type="aspectj" experssion="com.xxx.*Controller+"/>
</context:component-scan> 
```
过滤表达式：

|类别|实例|说明|
|:--:|:--|:--|
|annotation|com.jike.XxxAnnotation|符合XxxAnnotation的target class|
|assignable|com.jike.XxxService|指定父class或interface的全名|
|aspcetj|com.jike..*Service+|Aspectj语法，后缀为Service或Service的子类|
|regex|Com\.jike\.Default\.*|Regelar Experssion|
|custom|com.jike.myx|自定义|

### 常用注解详解
#### Bean定义注解
@Scope("prototype")作用域配置

1. @Repository
	将抛出的数据访问异常封装为spring的数据访问异常
	1. 使用@Repository将DAO类声明为Bean
	```java
	@Repository
	public class UserDAO{...}
	```
	2. 配置注解扫描包

2. @Component：泛化的概念
3. @Service
4. @Controller

#### Bean的生命周期
生命周期回调

* 实现Spring提供的两个接口：InitializingBean和DisposableBean
* 在XML文件中使用<bean>的init-method和destroy-method
	```xml
	<bean id="userService" class="com.jike.xxx.UserService"
		init-method="init" destory-method="destory"/>
	```

* JSR-250
	* @PostConstruct:初始化之后执行的回调方法
	* @PreDestory:销毁之前执行的回调方法
	* 配置文件示例说明：
		`<context:annotation-config/>`

#### 依赖检查注解
@Required注解，只能标注在setter之上。配置：`<context:annotation-config/>`
#### 自动装配注解
@Autowired可以对成员变量。方法和构造函数进行标注，根据类型进行自动装配，如需按名称进行装配，则需要配合@Qualifier使用，如下：
```java
@Service
public class logonService{
	@Autowired
	private LogDao logDao;
}
```
```java
@Service
public class logonService{
	@Autowired(require=false)
	private LogDao logDao;
}
```
```java
@Service
public class logonService{
	@Autowired
	@Qualifiler("userDao")
	private LogDao logDao;
}
```
方法入参自动装配
```java
@Service
public class logonService{
	@Autowired
	public void setLogDao(LogDao logDao){
		this.logDao = logDao;
	}
	@Autowired
	@Qualifier("userDao")
	public void setUserDao(UserDao userDao){
		this.userDao = userDao;
	}
}
```
```java
@Service
public class logonService{
	@Autowired
	public void setUserDao(@Qualifier("userDao")UserDao userDao){
		this.userDao = userDao;
	}
}
```
集合类自动装配标注，会将容器中类型匹配的所有Bean都自动注入进来。

## 基于Java类的配置
通过Java类定义Spring配置元数据，且直接消除XML配置文件：
### 基于Java类的配置示例
1. @Configuration注解需要作为配置的类
2. @Bean注解相应的方法
3. AnnotationConfigApplicationContext或子类进行加载

配置类示例：
```java
@Configuration
public class ApplicationContextConifg {
	@Bean
	public String message() {
		return "hello";
	}
}

	
public class ConfiguartionTest {
	public static void main(String[] args) {
		AnnotationConfigApplication ctx = 
		new AnnotationConfigApplicationContext(ApplicationContextConfig.class);
		System.out.println(ctx.getBean("message"));
	}
}
```

Bean注解格式
```java
@Bean(name={},
	autowire=Autowire.No,
	initMethod="",
	destoryMethod="")
```
### 结合使用java方式和xml
#### 基于java方法的配置类中引入基于xml方式的配置文件
配置文件：
```xml
<bean id="message" class="java.long.String">
	<constructor-arg index="0" value="test"/>
</bean>
```
```java
@Configration("ctxConfig")
@ImportResource("classpath:com/jike/***/appCtx.xml","")
public class ApplicationContextConfig{
	......
}
```
#### 在xml中引入基于java的配置
开注解扫描
```xml
<context:annotation-config/>
<bean id="ctxConfig" class="com.jike.***.ApplicationContextConfig"/>
```
### 启动Spring容器
* 通过构造函数加载配置类
```java
	ApplicationContext ctx = 
	new AnnotationConfigApplicationContext(AppConf.class);
```
* 手动注册
```java
	ApplicationContext ctx = 
	new AnnotationConfigApplicationContext();
	ctx.register(DaoConfig.class);
	ctx.register(ServiceConfig.class);
	ctx.refresh();
```
* 引入多个配置类
```java
	@Configuration
	@Import(DaoConfig.clcass)
	public class ServiceConfig{}
```

## 不同配置方式的比较
![比较](/img/比较.png)

基于XML的配置：
- 第三方类库，如DataSource，JdbcTemplate等
- 命名空间：如aop、context等

基于注解的配置：
- Bean的实现类是当前项目开发的，可直接在Java类中使用注解配置

基于Java类的配置：
- 对于实例化Bean的逻辑比较复杂，则比较适用于基于Java类配置的方式


