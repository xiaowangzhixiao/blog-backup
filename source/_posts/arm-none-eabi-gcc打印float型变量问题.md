---
title: arm-none-eabi-gcc打印float型变量问题
toc: true
comments: true
date: 2019-09-15 16:53:52
tags:
- 单片机
category:
- 编程
- stm32
---

在使用arm-none-eabi-gcc编译单片机程序时，发现使用printf，sprintf和vsnprintf之类的函数，无法正确序列化float型变量，即%f和%lf都无法正确处理，经过一番google，发现是在默认的情况下，arm-none-eabi-gcc不对浮点数类型进行解析。

![float输出的问题.png](/img/float输出的问题.png)

如何打开开关呢，在makefile文件中LDFLAGS添加链接选项`-u _printf_float`
```makefile
LDFLAGS = $(MCU) -specs=nano.specs -T$(LDSCRIPT) $(LIBDIR) $(LIBS) -Wl,-Map=$(BUILD_DIR)/$(TARGET).map,--cref -Wl,--gc-sections -u _printf_float
```

如果不使用make进行编译和链接呢，一种方法是在所用的ide配置中，寻找链接选项，然后添加那个选项，另一种是在main.c文件中，函数体之外加上一段汇编代码`asm(".global _printf_float");`
