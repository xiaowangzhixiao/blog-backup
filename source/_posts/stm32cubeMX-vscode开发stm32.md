---
title: stm32cubeMX+vscode开发stm32
toc: true
comments: true
date: 2019-08-14 11:53:07
tags:
- 单片机
category:
- 编程
- stm32
---

stm32单片机的开发使用哪个IDE比较好呢，这是这两年开发stm32程序以来一直在探索的。

一开始刚刚进入机器人队，学长教我们的是stm32库函数+keil开发程序，然后玩了一年的MDK，发现不怎么好用，而且keil没法跨平台啊，自己用linux开发单片机程序也想有IDE啊，纯vim+交叉编译工具链问题在于make难写啊，自己学make又记不住，vim也玩得不怎么好，，后来队友教下一届的用iar，，，试用了一下发现，代码补全还没keil来的好，怒弃，这个时候队里使用的底层已经转变成HAL库了，st公司也出了STM32cubeMX，底层配置只要点点点就可以了，点击生成代码就可以生成各家IDE的工程。

我在之前用vscode就发现vscode真不愧是宇宙最强编辑器，所以就开始用vscode写单片机代码，用iar下载和调试。前两天就在想，既然都用vscode写代码了，那直接用它下载调试不就行了，而且有stm32cubeMX，makefile可以自动生成，跨平台开发也实现了，经过一番探索，我成功实现了windows平台下在vscode编写和调试单片机代码，现记录如下。

<!-- more -->

## 安装vscode

正常安装vscode即可，不再赘述。

## 安装make工具

Windows系统没有make工具，需要下载mingw，mingw-w64-install.exe，安装好后将其bin目录加入到环境变量中

![mingw环境变量](/img/mingw环境变量.png)

这个时候在命令行里输入make -v发现还是找不到命令，这是怎么回事呢，原来是由于mingw中的make不叫make，打开mingw的bin目录将mingw32-make复制一份改名成make就可以了。

![make重命名](/img/make重命名.png)

![make](/img/make.png)

## 安装交叉编译工具链

我们要安装的是ARM GCC Toolchain (https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads)

安装好后添加环境变量，在命令行中验证： 
arm-none-eabi-gcc -v

![arm-gcc](/img/arm-gcc.png)

## 安装openocd

openocd(http://gnutoolchains.com/arm-eabi/openocd/) 是开源调试软件，连接仿真器后可以开GDB服务。

下载完之后是个压缩包，解压到安装位置后添加环境变量

命令行中验证

![openocd验证.png](/img/openocd验证.png)

## 安装clang llvm
安装clang主要是为vscode提供语法补全代码格式化等功能，安装好后添加环境变量

![环境变量总结.png](/img/环境变量总结.png)

## 生成并编译代码

在STM32CubeMX中配置好工程后选择makefile类型，然后生成代码。

在文件夹下命令行运行make就可以编译生成可以下载的二进制代码了。

## 配置vscode

### 安装插件
需要安装c/c++、ARM、Cortex-dubug，ARM插件用于格式化ARM汇编代码，Cortex-debug插件用于简化调试配置文件

![vscode插件.png](/img/vscode插件.png)

### 配置文件

用vscode打开工程文件夹，然后保存为工作区，在工程文件夹下新建文件.vscode\c_cpp_properties.json和.vscode\launch.json。
#### c_cpp_properties.json
.vscode\c_cpp_properties.json是用于对C/C++语言的语法提示等的配置，其中include路径和宏定义可以参照makefile添加。

```json
{
    "configurations": [
        {
            "name": "npc",
            "includePath": [
                "${workspaceFolder}/**",
                "D:/GNU Tools ARM Embedded/5.4 2016q3/**"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE",
                "USE_HAL_DRIVER",
                "STM32F103xB"
            ],
            "cStandard": "c99",
            "intelliSenseMode": "clang-x64",
            "browse": {
                "path": [
                    "${workspaceRoot}",
                    "${workspaceRoot}/Inc",
                    "${workspaceRoot}/Drivers/STM32F4xx_HAL_Driver/Inc",
                    "${workspaceRoot}/Drivers/CMSIS/Device/ST/STM32F4xx/Include",
                    "${workspaceRoot}/Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM4F",
                    "${workspaceRoot}/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS",
                    "${workspaceRoot}/Middlewares/Third_Party/FreeRTOS/Source/include",
                    "${workspaceRoot}/Drivers/CMSIS/Include",
                    "D:/GNU Tools ARM Embedded/5.4 2016q3/arm-none-eabi/include",
                    "D:/GNU Tools ARM Embedded/5.4 2016q3/arm-none-eabi/include/c++/5.4.1",
                    "D:/GNU Tools ARM Embedded/5.4 2016q3/arm-none-eabi/include/c++/5.4.1/arm-none-eabi/thumb/v7-m",
                    "D:/GNU Tools ARM Embedded/5.4 2016q3/arm-none-eabi/include/c++/5.4.1/backward",
                    "D:/GNU Tools ARM Embedded/5.4 2016q3/arm-none-eabi/include/sys",
                    "D:/GNU Tools ARM Embedded/5.4 2016q3/lib/gcc/arm-none-eabi/5.4.1/include",
                    "D:/GNU Tools ARM Embedded/5.4 2016q3/lib/gcc/arm-none-eabi/5.4.1/include-fixed"
                ],
                "limitSymbolsToIncludedHeaders": true,
                "databaseFilename": ""
            }
        }
    ],
    "version": 4
}
```

#### launch.json
这个文件主要是对调试任务进行配置，可以参考cortex-debug的官网完成。

```json
{
    "trace": true,
    "configurations": [
        {
            "cwd": "${workspaceRoot}",
            "executable": "./build/npc.elf",
            "name": "Debug stm32",
            "request": "launch",
            "type": "cortex-debug",
            "servertype": "openocd",
            "device": "STM32F103C8",
            "configFiles": [
                "./stm32f103c8.cfg"
            ],
            "svdFile": "./STM32F103.svd"
        }
    ]
}
```

文件中需要配置的地方有"executable": "./build/npc.elf"，改为刚刚编译成功的可执行文件

device改为自己芯片的类型

openocd连接单片机需要配置文件配置一下，就是`"configFiles": ["./stm32f103c8.cfg"]`，在工程根目录下新建一个配置文件

```bash
source [find interface/jlink.cfg]

#transport select hla_swd
transport select swd

#set WORKAREASIZE 0x4000
source [find target/stm32f1x.cfg]
```

还有一个就是svdFile，用于描述芯片外设的，配置后在调试时可以读取外设的寄存器，这个在网上找一下然后将相应类型的文件配置一下就可以。

## 下载和调试代码

在调试之前，需要一个叫做UsbDriverTool的工具，将下载器的驱动改为libusb，否则openocd无法连接到jlink

![libusb.png](/img/libusb.png)

配置好launch.json后，在vscode的调试界面就可以看到刚刚配置的debug，点击运行，就可以进行调试了

![vscode调试](/img/vscode调试.png)

