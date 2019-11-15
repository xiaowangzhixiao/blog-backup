---
title: LEX简介
date: 2018-03-17 13:45:40
tags: 
- 编译原理
- C语言
- linux
category: 
- 编程
- 编译原理
---

## 简介
Lex代表Lexical Analyzar，是一种生成扫描器的工具，可以识别文本中的词汇模式。
Lex与C是强耦合的，一个.l文件通过公用的LEX程序来传递，并生成c的输出文件。这些文件被编译为词法分析器的可执行版本。
lex把每个扫描出来的单词叫统统叫做token，token可以有很多类。对比自然语言的话，英语中的每个单词都是token，token有很多类，比如non(名词)就是一个类token，apple就是属于这个类型的一个具体token。对于某个编程语言来说，token的个数是很有限的，不像英语这种自然语言中有几十万个单词。
lex工具会帮我们生成一个yylex函数，yacc通过调用这个函数来得知拿到的token是什么类型的，但是token的类型是在yacc中定义的。
lex的输入文件一般会被命名成 .l文件，通过lex XX.l 我们得到输出的文件是lex.yy.c

<!-- more -->

## 常规表达式，正则表达式，re
### 用 Lex 定义常规表达式
|字符                      |含义                                                                      |
|:--------------------------|:---------------------------------------------------|
|A-Z,a-z,0-9|构成了部分模式的字符和数字|
|.|匹配任意字符，除了\n。|
|-|用来指定范围|
|[]|一个字符集合，匹配括号中的任意字符如果第一个字符是^那么它表示否定模式。|
|*|匹配0个或多个上述模式|
|+|匹配1个或多个上述模式|
|？|匹配0个或一个上述模式|
|$|作为模式的最后一个字符匹配一行的结尾|
|{x,y}|指出一个模式可能出现的次数，例如: A{1,3} 表示 A 可能出现1次到3次。|
|\|用来转义元字符|
|^|用来表示否定|
|&#124;|表达式间的逻辑或|
|"<一些符号>"|字符的字面含义。元字符具有。|    ???
|/|向前匹配。如果在匹配的模版中的“/”后跟有后续表达式，只匹配模版中“/”前 面的部分。如：如果输入 A01，那么在模版 A0/1 中的 A0 是匹配的。|
|()|将一系列正则表达式分组|

### 举例
|表达式|含义|
|:--|:--|
|joke[rs] |匹配 jokes 或 joker。
|A{1,2}shis+| 匹配 AAshis, Ashis, AAshiss, Ashiss，Ashisss,...
|(A[b-e])+ |  匹配在 A 出现位置后跟随的从 b 到 e 的所有字符中的 0 个或 1个。

### 正则式

number [0-9]+

id          [A-Za-z]+[A-Za-z0-9_]*

## Lex编程

1. 以Lex可以理解的格式指定模式相关是动作。
2. 在这一文件上运行Lex，生成扫描器的C代码。
3. 编译链接C代码生成可执行的扫描器。

一个 Lex 程序分为三个段：第一段是 C 和 Lex 的全局声明，第二段包括模式（C 代码），第三段是补充的 C 函数。 例如, 第三段中一般都有 main() 函数。这些段以%%来分界。
```
        Definition section
        %%
        Rules section
        %%
        C code section
```
- Definition Section
    这块可以放C语言的各种各种include，define等声明语句，但是要用%{ %}括起来。
    .l文件，可以放预定义的正则表达式：minus "-" 还要放token的定义，方法是：代号 正则表达式。然后到了Rules Section就可以通过{符号} 来引用正则表达式
```
        %{
        int wordCount = 0;
        %}
        chars [A-za-z\_\'\.\"]
        numbers ([0-9])+
        delim [" "\n\t]
        whitespace {delim}+
        words {chars}+
        %%
```
- Rules section
    .l文件在这里放置的rules就是每个正则表达式要对应的动作，一般是返回一个token
    这里的动作都是用{}扩起来的，用C语言来描述，这些代码可以做你任何想要做的事情
```
        {words} { wordCount++; /*
        increase the word count by one*/ }
        {whitespace} { /* do
        nothing*/ }
        {numbers} { /* one may
        want to add some processing here*/ }
        %%
```
- C code Section
    main函数
```c
        void main()
        {
            yylex(); //这一函数开始分析。 它由 Lex 自动生成。
            printf(" No of words: %d\n", wordCount);
        }
        /*这一函数在文件（或输入）的末尾调用。 如果函数的返回值是1，就停止解析。
         因此它可以用来解析多个文件。 代码可以写在第三段，这就能够解析多个文件。
         方法是使用 yyin 文件指针指向不同的文件，直到所有的文件都被解析。 
         最后，yywrap() 可以返回 1 来表示解析的结束。*/
        int yywrap()
        {
            return 1;
        }
```

## 编译运行

### 运行环境
ubuntu15.10
lex/flex 本机预安装了flex，使用lex命令和flex命令相同
gcc

### 编译
```
lex file.l                      #生成lex.yy.c
gcc lex.yy.c -o yourname -ll    #编译生成可执行程序 -ll 用来链接lex库
./youname < xxx.c               #运行程序，将输入流重定向为要分析的文件
```


