---
title: vscode+jflash开发调试stm32
toc: true
comments: true
date: 2019-09-10 20:12:55
tags:
- 单片机
category:
- 编程
- stm32
---

下面介绍一下windows下使用vscode和jflash对stm32开发
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

## 安装jflash
正常安装jflash，一般安装jlink的驱动时会安装，然后需要添加环境变量

## 安装clang llvm
安装clang主要是为vscode提供语法补全代码格式化等功能，安装好后添加环境变量

![环境变量2.png](/img/环境变量2.png)

## 生成并编译代码

在STM32CubeMX中配置好工程后选择makefile类型，然后生成代码。

在文件夹下命令行运行make就可以编译生成可以下载的二进制代码了。

## 配置vscode

### 安装插件
需要安装c/c++、ARM、Cortex-dubug，ARM插件用于格式化ARM汇编代码，Cortex-debug插件用于简化调试配置文件

![vscode插件.png](/img/vscode插件.png)

### 配置文件

用vscode打开工程文件夹，然后保存为工作区。
#### c_cpp_properties.json
使用ctrl+shift+p,输入命令 `C/C++:Edit Configurations(UI)`,打开配置界面
![configuration.png](/img/configuration.png)
- 添加新配置
- 添加arm-none-eabi-gcc的编译路径
- 打开makefile文件，将其中的宏定义添加到配置界面
- c语言版本选择c99
- 打开.vscode\c_cpp_properties.json文件将c++版本的配置删除

![CompilerPath.png](/img/CompilerPath.png)
![Defines.png](/img/Defines.png)

.vscode\c_cpp_properties.json是用于对C/C++语言的语法提示等的配置，配置完成后.vscode\c_cpp_properties.json如下
```json
{
    "configurations": [
        {
            "name": "npc",
            "includePath": [
                "${workspaceFolder}/**",
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE",
                "USE_HAL_DRIVER",
                "STM32F103xB"
            ],
            "compilerPath": "\"D:/GNU Tools ARM Embedded/5.4 2016q3/bin/arm-none-eabi-gcc.exe\"",
            "cStandard": "c99",
            "intelliSenseMode": "clang-x86"
        }
    ],
    "version": 4
}
```

#### launch.json
这个文件主要是对调试任务进行配置，使用ctrl+shift+p,输入命令 `Debug:Open launch.json`，选择Cortex Debug,如下
![debug1.png](/img/debug1.png)
![debug2.png](/img/debug2.png)
```json
{
    "trace": true,
    "configurations": [
       {
            "cwd": "${workspaceRoot}",
            "executable": "./build/npc.elf",
            "name": "Debug Microcontroller",
            "request": "launch",
            "type": "cortex-debug",
            "device": "STM32F103C8",
            "servertype": "jlink",
            "svdFile": "./STM32F103.svd"
        }
    ]
}
```

文件中需要配置的地方有
- "executable": "./build/npc.elf"，改为刚刚编译成功的可执行文件
- device改为自己芯片的类型
- svdFile，用于描述芯片外设的，配置后在调试时可以读取外设的寄存器
## 下载和调试代码

配置好launch.json后，在vscode的调试界面就可以看到刚刚配置的debug，点击运行，就可以进行调试了

![vscode调试](/img/vscode调试.png)



