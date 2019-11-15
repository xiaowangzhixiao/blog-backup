---
title: Java虚拟机运行时内存模型
date: 2018-09-30 22:52:17
tags:
- JVM
category:
- 编程
- java
---

<body>
<h1 align="center" class="root">
<a name="407os5qi6nrv7kpt5n6emn3j0s">java虚拟机运行时内存</a>
</h1>
<div align="center" class="globalOverview">
<img src="/img/java%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%BF%90%E8%A1%8C%E6%97%B6%E5%86%85%E5%AD%98.png"></div>

<!-- more -->

<h2 class="topic">
<a name="6jb9bh6jbo3e9vnt37gtre3n7e">程序计数器</a>
</h2>
<h2 class="topic">
<a name="1nfkogcbgifho8451cssouolj1">虚拟机栈</a>
</h2>
<div class="overview">
<img src="/img/%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%A0%88 2.jpg"></div>
<h3 class="topic">
<a name="7v8phq556po1su30es98oif8s0">&nbsp;局部变量表</a>
</h3>
<h3 class="topic">
<a name="4troi8h8248rkc5shkf2mtce74">&nbsp;操作数栈</a>
</h3>
<h3 class="topic">
<a name="3auelu6qun19c2ij7vepbpfqid">&nbsp;动态链接</a>
</h3>
<h3 class="topic">
<a name="6r3h8gi9b526pshdkvakknsfm9">&nbsp;方法出口</a>
</h3>
<h2 class="topic">
<a name="7jf17jn05v8crp092tnhif1g85">本地方法栈</a>
</h2>
<h2 class="topic">
<a name="7o7ktgujjooqd8m6pkc3anet3g">Java堆</a>
</h2>
<div class="overview">
<img src="/img/Java%E5%A0%86 2.jpg"></div>
<h3 class="topic">
<a name="396fhu2q4pig7ehmca8hd1eld0">&nbsp;新生代</a>
</h3>
<div class="overview">
<img src="/img/%E6%96%B0%E7%94%9F%E4%BB%A3 2.jpg"></div>
<h3 class="topic">
<a name="1e8oe6nkjm8j5le9vre2ie7kf0">&nbsp;&nbsp;Eden</a>
</h3>
<h3 class="topic">
<a name="0hlgqru77v8h79epuntouu949a">&nbsp;&nbsp;From Survivor</a>
</h3>
<h3 class="topic">
<a name="3c9b3o82heuls4pgilvr68038s">&nbsp;&nbsp;To Survivor</a>
</h3>
<h3 class="topic">
<a name="14ahpn4ts5tl21at550qbvtqbf">&nbsp;老年代</a>
</h3>
<h3 class="topic">
<a name="6la97h8tdgtn1tt76vld5d4k5r">&nbsp;线程私有分配缓冲区TLAB</a>
</h3>
<h2 class="topic">
<a name="7md98oqslraoairt3sru65bpf2">方法区</a>
</h2>
<div class="overview">
<img src="/img/%E6%96%B9%E6%B3%95%E5%8C%BA 2.jpg"></div>
<h3 class="topic">
<a name="34d572fiff1i0mrqprepkm1ot8">&nbsp;类信息</a>
</h3>
<h3 class="topic">
<a name="7j9hr9hpbbo7tc8mpf0vulslp7">&nbsp;常量</a>
</h3>
<h3 class="topic">
<a name="1qmpr86ukis1ldh39f98llj1i8">&nbsp;静态变量</a>
</h3>
<h3 class="topic">
<a name="2kedhq37oincbad5fsfqj82b5l">&nbsp;即时编译器编译后的代码</a>
</h3>
<h3 class="topic">
<a name="6j1h6ahc13oqpp5akm8dnqtb01">&nbsp;运行时常量池</a>
</h3>
</body>
</html>

