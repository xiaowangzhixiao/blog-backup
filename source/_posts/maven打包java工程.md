---
title: maven打包java工程
date: 2018-07-16 22:56:38
tags:
- maven
category:
- 编程
- Java
---

使用maven管理java工程，maven的生命周期中有一个重要的package阶段，若是Java Web项目，在pom文件中将知名打包为war包，而若是java工程，则一般打包为jar包，但是如果直接执行maven的package操作时，会只将项目代码打包为jar，而不会带着依赖，单个jar包无法跑起来，下面就介绍两种使用maven打包依赖的方式。

<!--more-->

## 打包为带依赖的jar包

使用两个maven插件可以将依赖和源码一起打包到jar包中

### assembly 
在pom文件插件部分加入如下配置：
```xml
	<plugin>
	    <artifactId>maven-assembly-plugin</artifactId>
	    <configuration>
	        <archive>
	            <manifest>
	            	<!-- 程序主类入口，加上可直接用java -jar xxx.jar命令运行 -->
	                <mainClass>${exec.mainClass}</mainClass>
	            </manifest>
	        </archive>
	        <descriptorRefs>
	            <descriptorRef>jar-with-dependencies</descriptorRef>
	        </descriptorRefs>
	    </configuration>
	</plugin>
```
运行maven的assembly插件即可将代码打包为带依赖的jar包，在target文件夹下可以看到xxx.jar为package阶段打包的，xxx-jar-with-dependencies.jar则为带依赖的jar包
上面`<descriptorRefs>`是官方提供的定制化打包方式，其中有 bin, jar-with-dependencies, src, project这几种配置，具体分别代表什么之后会有更新，一般不推荐使用。
推荐使用自定义打包方式，配置如下：
```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-assembly-plugin</artifactId>
	<version>2.2-beta-4</version>
	<configuration>
		<descriptors>
			<descriptor>src/main/assembly/src.xml</descriptor>
		</descriptors>
		<archive>
			<manifest>
				<mainClass>${exec.mainClass}</mainClass>  
			</manifest>
		</archive>
	</configuration>
</plugin>
```
在src.xml文件中指定了打包操作:
```xml
<assembly
        xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.1"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.1 http://maven.apache.org/xsd/assembly-1.1.1.xsd">
    <id>jar-with-dependencies</id>
    <formats>
        <format>jar</format>
    </formats>

    <includeBaseDirectory>false</includeBaseDirectory>
    <dependencySets>
        <dependencySet>
            <unpack>true</unpack>
            <scope>runtime</scope>
        </dependencySet>
    </dependencySets>
    <fileSets>
        <fileSet>
        	<!-- 资源文件输出 -->
            <directory>src\main\resources</directory>
            <outputDirectory>\</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>${project.build.outputDirectory}</directory>
        </fileSet>
    </fileSets>
</assembly>
```
### shade
通过shade插件我们可以为生成的那个jar包选择包含哪些依赖以及排除哪些依赖。 
1. 支持两种操作include和exclude 
2. 配置格式：groupId:artifactId[[:type]:classifier]，至少包含groupid和artifactid，type和类名可选 
3. 支持’\*’ 和 ‘?’执行通配符匹配 

shade插件打包时在对spring.schemas文件处理上，它能够将所有jar里的spring.schemas文件进行合并，在最终生成的单一jar包里，spring.schemas包含了所有出现过的版本的集合

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <filters>
                    <filter>
                        <artifact>:</artifact>
                        <excludes>
                            <exclude>META-INF/*.SF</exclude>
                            <exclude>META-INF/*.DSA</exclude>
                            <exclude>META-INF/*.RSA</exclude>
                        </excludes>
                    </filter>
                </filters>
                <!-- 默认值为true.注意这个属性,如果你用这个插件来deploy,或者发布到中央仓库，这个属性会缩减你的pom文件,会把你依赖的<dependency>干掉 -->
                <createDependencyReducedPom>false</createDependencyReducedPom>
                <transformers>
                    <transformer
                            implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <!-- 这个是你的程序入口文件 -->
                        <mainClass>com.timer.CollectBuptBBS</mainClass>
                    </transformer>
                    <transformer
                            implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                        <resource>META-INF/spring.handlers</resource>
                    </transformer>
                    <transformer
                            implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                        <resource>META-INF/spring.schemas</resource>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>

```

执行mvn package，生成两个jar文件，一个是原始的original-xxx-1.0-SNAPSHOT.jar，一个是可执行的xxx-1.0-SNAPSHOT.jar。

## 打包为外部依赖
在pom文件中进行如下配置：
```xml
<!-- 打包jar文件时，配置manifest文件，加入lib包的jar依赖 -->
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-jar-plugin</artifactId>
	<configuration>
		<archive>
			<manifest>
				<addClasspath>true</addClasspath>
				<classpathPrefix>lib/</classpathPrefix>
				<mainClass>com.timer.CollectBuptBBS</mainClass>
			</manifest>
		</archive>
	</configuration>
</plugin>
 <!--拷贝依赖的jar包到lib目录-->
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-dependency-plugin</artifactId>
	<executions>
		<execution>
			<id>copy</id>
			<phase>package</phase>
			<goals>
				<goal>copy-dependencies</goal>
			</goals>
			<configuration>
				<outputDirectory>
					${project.build.directory}/lib
				</outputDirectory>
			</configuration>
		</execution>
	</executions>
</plugin>
```
执行mvn package,target文件夹中产生jar文件和lib文件夹，jar为源码打包的文件，内部指明了主类和依赖的路径是lib文件夹，lib文件夹中为依赖的jar包，在运行环境中需要将lib和jar文件放到同一路径下执行java -r xxx.jar即可运行。