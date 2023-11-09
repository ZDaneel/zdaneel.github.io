---
title: "xv6在m1mac上安装以及在vscode里debug.md"
date: 2023-11-09T15:50:36+08:00
draft: false
toc: true
---


## 环境：

电脑型号：MacBook Air (M1, 2020)

系统版本：macOS Sonoma 14.0

## 一、安装

> 参考https://zhuanlan.zhihu.com/p/464386728

### riscv-gnu-toolchain

使用链接中的2源码编译安装，其百度网盘文件编译没有产生问题

### qemu

QEMU emulator version 8.1.2

```shell
brew install qemu
```

## 二、vscode调试

### xv6启动

在xv6目录下使用`make CPUS=1 qemu-gdb`，记住最后的端口号

![image-20231109220205651](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202311092204946.png)

### 配置文件

#### 1. .gdbinit

> mac下查看隐藏文件快捷键`cmd+shift+.`

在主目录`~`下创建一个`.gdbinit`,从而使得gdb启动时能够读取xv6目录下的`.gdbinit`

初始化gdb，从而无需在每次打开gdb时重复配置

 ```shell
 add-auto-load-safe-path {YourPath}/xv6-riscv/.gdbinit
 ```

#### 2. launch.json和tasks.json

在xv6目录下的`.vscode`文件夹中创建两个文件，注意两个都是复数

launch.json

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/kernel/kernel",
            "cwd": "${workspaceFolder}",
            "miDebuggerPath": "/opt/riscv-gnu-toolchain/bin/riscv64-unknown-elf-gdb", // 可能需要更改
            "miDebuggerServerAddress": "127.0.0.1:YourPort", // 修改端口
            "MIMode": "gdb",
            "stopAtEntry": true,
            "preLaunchTask": "fix_gdbinit",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": false
                }
            ]
        }
    ]
}
```

tasks.json

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "fix_gdbinit",
            "type": "shell",
            "command": "sed -i '' -e '/^target remote/d' ${workspaceFolder}/.gdbinit"
        }
    ]
}
```

之后启动vscode的debug，如果一切正常，如下图所示。就可以欢乐调试了。

![image-20231109221637470](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202311092216400.png)

## 三、其他

推荐绿老师的介绍视频 https://www.bilibili.com/video/BV1DY4y1a7YD

踩坑点：make qemu之后，**按下ctrl+a后松开**，然后按下

- x是退出
- c是进程qemu控制台

在qemu控制台中，可以通过`info mem`、`info registers`等指令查看模拟硬件的内容

再次按下ctrl+a后松开，然后按下c就可以退出qemu控制台
