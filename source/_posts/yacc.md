---
title: yacc简介
date: 2018-03-17 14:29:30
tags: 
- 编译原理
- C语言
- linux
category: 
- 编程
- 编译原理
---

## 简介
Yacc是一个工具，它将任何一种编程语言的所有语法翻译成针对此钟语言的Yacc语法解析器。
它用巴克斯范式（BNF）来书写
## 步骤
1. 运行yacc处理语法文件，生成一个解析器
2. 说明语法：
    -编写.y语法文件
    -编写一个词法分析器来处理输入并将标记传递给处解析器。这里可以使用Lex完成。
    -编写一个函数，使用yyparse()开始解析
    -编写错误处理例程 如yaerror()
3. 编译yacc生成的代码以及其他相关源文件
4. 将目标文件链接到适当的可执行解析器库

<!-- more -->

## 编写
和LEX类似，yacc的程序也分为三段
```
        declarations
        %%
        rules
        %%
        programs
```
### rules
yacc中最重要的是其中的文法规则，首先来看文法如何书写
yacc文法使用BNF巴克斯范式书写
如下：
```
        result: components { /*
        action to be taken in C */ }
            | components
            |...
        ;
```
{}中书写识别出该表达式所做的动作
例：
```
        param : NAME EQ NAME {
            printf("\tName:%s\tValue(name):%s\n", $1,$3);}
            | NAME EQ VALUE{
            printf("\tName:%s\tValue(value):%s\n",$1,$3);}
        ;
```
其中$n是使用了产生式中第n个标记的值，lexer通过yylval返回值。$$表示规约后的值。

标记 NAME 的 Lex 代码是这样的：
```
       char [A-Za-z]
        name {char}+
        %%
        {name} { yylval = strdup(yytext); //strdup函数是使用malloc函数进行字符串的复制
        return NAME; }
```
### declarations
与LEX类似，使用C语言的声明，需要用
```
    %{
       balabala
    %}
```
来声明，包括include等等，其中需要定义YYSTYPE，它是内置变量yylval的类型，默认为int，例如：
```
    %{
        #typedef char* string; /*
        to specify token types as char* */
        #define YYSTYPE string /*
        a Yacc variable which has the value of returned token */
    %}
```
用户经常会希望语义值的类型比较复杂，如双精度浮点数，字符串或树结点的指针．这时就可以用语义值类型定义进行说明。因为不同的语法符号的语义值类型可能不同，所以语义值类型说明就是将语义值的类型定义为一个联合（Union），这个联合包括所有可能用到的类型（各自对应一个成员名），为了使用户不必在存取语义值时每次都指出成员名，在语义值类型定义部分还要求用户说明每一个语法符号（终结符和非终结符）的语义值是哪一个联合成员类型。
```
    ％ union｛
    int ival
    double dval
    INTERVAL VVal;
    ｝
    ％token ＜ival＞ DREG VREG
    ％token ＜dval＞ CONST
    ％type ＜dyal＞dexp
    ％type ＜vval＞vexP
```
使用%union来定义一个联合类型，作为yylval的类型，当声明TOKEN和非终结符时，在中间加<...>来声明语义值的类型，非终结符不需要声明语义值时，不用刻意声明

声明终结符，即lexer返回的TOKEN：
```
    %token NAME EQ AGE
```
每个 Yacc 声明段声明了终端符号和非终端符号（标记）的名称，还可能描述操作符优先级和针对不同符号的数据类型。
```
    % start 非终结符……//声明文法起始符
```
如果没有上面的说明，yacc自动将语法规则部分中第一条语法规则左部的非终结符作为语法开始符。
```
    %left '+' '-'
    %left '*' '/'
```
%left 表示左结合，%right 表示右结合。最后列出的定义拥有最高的优先权。因此乘法和除法拥有比加法和减法更高的优先权。+ - * / 所有这四个算术符都是左结合的。运用这个简单的技术，我们可以消除文法的歧义。

在表达式中有时要用到一元运算符，而且它可能与某个二元运算符是同一个符号，例如一元运算符负号“-”就与减号’-’相同，显然一元运算符的优先级应该比相应的二元运算符的优先级高。至少应该与\*的优先级相同，这可以用yacc的％Prec子句来定义，请看下面的文法：
```
    ％token NAME
    ％left '-''+’
    ％left '*''／'
    ％％
    expr；expr'+' expr
    |expr'+' expr
    |expr'-’expr
    |expr'*' expr
    |expr'／’ expr
    |'-'expr ％prec'*'
    |NAME
    ;
```
在上述文法中，为使一元’－’的优先级与\*相同，我们使用了子句
```
    ％prec'*'
```
它说明它所在的语法规则中最右边的运算符或终结符的优先级与％Prec后面的符号的优先级相同，注意％Prec子句必须出现在某语法规则结尾处分号之前，％prec子句并不改变’－’作为二元运算符时的优先级。
### programs

```

        void main()
        {
            yyparse();
        }
        int yyerror(char* msg)  //用户可以自己实现错误函数
        {
        	printf("Error: %s
        	encountered \n", msg);
    	}
```
## 参考
[LEX和YACC的使用](http://blog.csdn.net/jiary5201314/article/details/8262134)
[Yacc 与 Lex 快速入门](https://www.ibm.com/developerworks/cn/linux/sdk/lex/)

