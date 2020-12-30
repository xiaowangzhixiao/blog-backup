---
title: Spring配置文件浅析
date: 2018-08-21 19:37:55
tags:
- Spring
- XML
category:
- 编程
- Java
---

Spring配置文件是用于指导Spring工程进行Bean的生产、依赖关系注入及Bean实例分发的“图纸”
<!-- more -->
## 配置文件实例
```xml
<beans>
    <!--import用于引入其他配置文件-->
    <import resource="resource1.xml"/>
    
    <!--bean用于定义bean实例-->
    <bean id="bean1" class="xxx1"/>
    <bean id="bean2" class="xxx2">
        <property name="hehe" ref="bean1" />
    </bean>
    
    <!--alias用于定义别名-->
    <alias alias="bean3" name="bean2"/>
    
</beans>

```
## 容器高层视图
Spring启动的基本条件：
* Spring框架依赖
* Bean的配置信息
* Bean的实现类

### Bean的元数据信息
* 实现类
* 属性信息
* 依赖关系
* 行为配置
* 创建方式---构造器还是工厂方法实例化



## 命名空间
使用xmlns schemas实现命名空间
## xml schema头配置信息
一个标准的Spring xml配置文件头如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" 
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="bean1" class="com.xiaowangzhixiao.helloImp.HelloChina"/>
</beans>
```
其中xmlns后没有任何命名的是默认的命名空间，声明Bean时未使用命名空间时的默认空间
xmlns:xsi是标准组织定义标准命名空间，为xml每个命名空间中指定相对应的schema
xsi:schemaLocation中为每个命名空间指定schema的url
xmlns:hahaha定义了hahaha命名空间
在schemaLocation中定义xsd的位置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    ...

</beans>
```
### schema命名空间和schema文件对应关系
- 命名空间： `http://www.springFramework.org/schema/beans`
- Schema文件：`http://www.springFramework.org/schema/beans/spring-beans.xsd`
schema文件可以加版本号例如spring-beans-5.0.0.xsd

### 常用schema
* beans.xsd
* aop
* tx    事务
* mvc
* util  简化标准配置
* jee   EJB，JNDI
* jdbc
* jms   
* lang  集成动态语言
* oxm   配置对象映射到shema
* task  任务调度
* tool  集成Spring有用的工具

## bean
### bean的id
命名方式
* 配置全限定类名，只配置类，唯一
* id
* name
* id和name
* 指定多个name 用分号分隔
* 指定别名
id命名规范
* 遵循xml命名规则
* 字母数字和下划线组成
* 驼峰式

### 实例化
#### 构造器实例化
1. 空构造器实例化：
    ```xml
    <bean id="bean1" class="HelloImpl"/>
    ```
2. 有参数构造器实例化
    ```xml
    <bean id="bean1" class="HelloImpl">
        <!--指定构造器参数-->
        <constructor-arg index="0" value="Hello Spring"/>
    </bean>
    ```

#### 静态工厂实例化
```xml
<bean id="bean" class="HelloFactory" factory-method="newInstance">
    <!--指定构造器参数-->
    <constructor-arg index="0" value="Hello Factory"/>
</bean>
```

#### 实例工厂实例化
```xml
<beans>
    <!--1. 定义实例工厂Bean-->
    <bean id="helloFactory" class="HelloFactory"/>
    <bean id="hello" factory-bean="beanInstanceFactory" factory-method="newInstance">
        <constructor-arg index="0" value="Hello Spring"/>
    </bean>
</beans>
```

### bean的作用域
scope，创建的Bean对象相对于其他Bean对象的请求可及范围
#### 作用域类型与配置
1. singleton： `<bean id="userInfo" class="xxx" scope="singleton"/>` 缺省，每次请求Bean时，返回同一个实例对象
2. prototype：`<bean id="userInfo" class="xxx" scope="prototype"/>`  每次请求Bean时，重新创建一个新Bean，默认情况下，Spring容器在启动时不实例化prototype的Bean

使用Spring的WebApplicationContext时，可以使用以下几种作用域
3. request：针对每次Http请求创建Bean`<bean id="userInfo" class="xxx" scope="request"/>` 
4. session：Http Session `<bean id="userInfo" class="xxx" scope="session"/>` 
5. global session

在使用Web应用环境相关的作用域时，必须在Web容器中进行一些额外的配置
* 低版本的Web容器配置
```xml
<filter>
    <filter-name>requestContestFilter</filter-name>
    <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>requestContestFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```
* 高版本的Web容器配置
```xml
<listener>
<listener-class>
org.springframework.web.context.request.RequestContextListener
</listener-class>
</listener>
```

#### 自定义作用域
* 实现自定义Scope类
```
org.springframework.beans.factory.config.Scope
```
* 注册自定义类
```
ConfigutableBeanFactory.registerScope(String scopename,Scope scope);
```
* 使用Scope类
