---
title: "同步时序电路分析"
date: 2022-07-27T15:50:36+08:00
katex : true
draft: false
toc: true
---



# 新手上路实验

## 一、概述

[下载地址](https://github.com/Logisim-Ita/Logisim)

使用的是Logisim的Ita版本，支持简体中文

[学习视频地址](https://www.icourse163.org/learn/HUST-1205809816?tid=1450219449#/learn/announce)

## 二、案例

### 1. LED计数电路

首先连接好电路

<img src="https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202203251613313.png" alt="截屏2022-03-25 16.13.00" style="zoom: 33%;" />

之后对电路进行封装：

<img src="https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202203251613858.png" alt="截屏2022-03-25 16.13.00" style="zoom: 33%;" />

再用封装好的进行ce shi

<img src="https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202203251615407.png" alt="截屏2022-03-25 16.15.34" style="zoom:33%;" />

### 2. 五输入编码器

在菜单「工程」中点击「分析组合逻辑电路」，之后就可以对输入输出、表达式、真值表等进行一定操作，使用表达式生成电路

![截屏2022-07-29 11.14.51](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202207291115284.png)

同样进行封装

<img src="https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202203251618986.png" alt="截屏2022-03-25 16.18.48" style="zoom:33%;" />

再用之前的电路进行测试，此处使用分线器将三个单独位的转成一个三位的二进制数，使用探针可以读取线路上的数据，可以用二进制、有/无符号十进制、十六进制等表示。



<img src="https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202203251619195.png" alt="截屏2022-03-25 16.19.32" style="zoom:33%;" />

### 3. 七段数码管

采用真值表的方式创建电路

<img src="https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202203251631374.png" alt="截屏2022-03-25 16.31.00" style="zoom:33%;" />

同样进行封装， 再进行测试，注意数码管的连接方式

<img src="https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202203251633802.png" alt="截屏2022-03-25 16.33.49" style="zoom:33%;" />

# 数据表示实验

## 一、国际转区位码

区位码的十六进制表示+2020H=国标码

题目要求用加法器来实现，加法器需要转换成补码形式

区位码=国际码-2020H+FFFFH+00001H

区位码=国际码+DFE0H

![截屏2022-04-19 22.50.52](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202204192250612.png)

## 二、偶检验

### 1. 偶检验编码

两两异或，实际上就是对全部的位进行异或，也可以只使用一个异或门

![截屏2022-04-20 10.44.21](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202204201044075.png)

### 2. 偶检验检错

就直接用了一个异或门

右上角不用去管，用分线器把p1信号已经分走了

![截屏2022-07-29 10.45.15](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202207291045620.png)

### 3. 测试

检错用于发送和接收双方之间，如果接受信号的发现有错误，就会提示发送方重新发送数据。

左边显示的是原来的数据，也就是发送方的数据。右边显示的就是接受方的数据，中间有一个干扰屏蔽字，是随机生成的。分别有三个提示信号

- 数据正确：干扰屏蔽字为00000，也就是没有错误
- 检错位：检查到了错误，由于偶检验只能检验出奇数个错误，偶数情况下检错位不会亮起
- 误报：针对的就是偶数位情况下的检错位无效，干扰前后两个信号是不同的，于是就误报了

![截屏2022-04-20 10.51.52](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202204201051132.png)

## 三、海明码

校验位：P1、P2、P3、P4、P5

被检验的数据位：D1、D2、D3……D15、D16

![image-20220420110856045](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202204201108399.png)

| 检验位 | 被检验的数据位                    |
| ------ | --------------------------------- |
| P1     | D1 D2 D4 D5 D7 D9 D11 D12 D14 D16 |
| P2     | D1 D3 D4 D6 D7 D10 D11 D13 D14    |
| P3     | D2 D3 D4 D8 D9 D10 D11 D15 D16    |
| P4     | D5 D6 D7 D8 D9 D10 D11            |
| P5     | D12 D13 D14 D15 D16               |

# ALU实验

## 一、1位全加器

由半加器变化而来，两个加数与一个进位输入，输出和还有进位

<img src="https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202204011328512.png" alt="截屏2022-04-01 13.27.58" style="zoom:67%;" />

## 二、8位串行进位可控加减法器

### 1. 8位加法器

首先来解决8位加法器的实现

OF溢出：针对带符号数，对最高数据位进位和符号位进位进行检测。

符号位产生进位为C0，最高位产生的进位为C1

逻辑表达式为：OF = C0⊕C1

异或门

|  C0  |  C1  |  OF  |
| :--: | :--: | :--: |
|  0   |  0   |  0   |
|  0   |  1   |  1   |
|  1   |  0   |  1   |
|  1   |  1   |  0   |

![截屏2022-04-01 13.45.56](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202204011346777.png)

### 2. 8位可控加减法器

运算公式

- $[A + B]_补 = [A]_补 + [B]_补$
- $[A - B]_补 = [A]_补 + [-B]_补$

加减法统一采用加法来处理

实现减法的核心在于如何求 $[-B]_补$

 $ [-B]_补 = \bar{B} + 1$

- +1只需要输入Cin
- 对Y进行取反，只需要一个异或门——sub为1时才对Y取反

|  Y   | Sub  | res  |
| :--: | :--: | :--: |
|  0   |  0   |  0   |
|  0   |  1   |  1   |
|  1   |  0   |  1   |
|  1   |  1   |  0   |

![截屏2022-04-01 14.12.28](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202204011412198.png)

串行电路最大的问题是，进位按串行传递，速度慢，n位串行加法器从C0到Cn到延迟位2n级门延迟

## 三、并行进位加法器

如何产生先行进位？

### 1.CLA72182

基本原理

[先行进位加法器 - oceany - 博客园](https://www.cnblogs.com/haigege/archive/2011/09/30/2196275.html)

按照推导出来的公式构建如下的电路 

![截屏2022-07-29 10.49.01](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202207291049501.png)

### 2. 4位快速加法器

Gi = XiYi

Pi = Xi + Yi（逻辑上实现加法，10得1，01得1，00得0， 11得0，所以采用异或门）![截屏2022-04-01 15.50.18](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202204011550525.png)

### 3. 16位快速加法器

![截屏2022-04-01 16.00.44](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202204011600344.png)

### 4. 32位快速加法器

<img src="https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202204011612666.png" alt="截屏2022-04-01 16.12.13" style="zoom:67%;" />

# 单周期CPU

## 一、概述

参考文章：24条指令MIPS单周期CPU - 计算机系统硬件设计 - [知乎专栏](https://zhuanlan.zhihu.com/p/156818024)

总体图采用华科老师的图，单周期也是最简单最基础的一个实现了，老师给的通路图是8条指令的实现，实际的电路多个几个指令。

![截屏2022-05-30 19.30.08](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202205301934794.png)

## 二、数据通路

一点一点开始连线

### 1. PC+4

使用ctrl+t后进行单步时钟测试，加法器进行加4

![截屏2022-05-30 19.49.36](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202205301949280.png)

### 2. 连接指令存储器

如果直接连接指令存储器，会发现位宽不匹配，原因在于指令存储器为地址10位，数据位宽为32位的ROM，所以需要从PC里取出2-11位的数据，使用一个分线器即可。

![截屏2022-05-30 19.57.33](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202205301957630.png)

此时可以进行测试，随着PC的运行，可以正常的取指令

![截屏2022-05-30 19.57.33](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202205302001084.png)

### 3. 译码

即使用分线器，将指令的不同位连接到寄存器组、控制器和ALU上

#### 3.1 R型指令

最简单的一个连线

![截屏2022-05-30 20.11.28](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202205302011843.png)

#### 3.2 I型指令

首先处理lw和sw指令，具体原理不再阐述，其中把复位键单独拿出来做成了一个Tunnel，之前的PC复位也改了一下，这样就可以一个按钮控制所有的复位。

![截屏2022-05-30 20.41.16](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202205302041213.png)

接下去处理的是Beq、Bne指令的数据通路，需要增加分支

红线是因为还没有处理控制器

![截屏2022-05-30 21.07.32](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202205302107963.png)

还需要考虑与立即数的加法，即对立即数进行零拓展

![截屏2022-05-30 21.46.23](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202205302146855.png)

#### 3.3 J型指令

Jmp、J、Jal（这部分老师没细讲，不太熟练）

Jmp连接的是控制器的Jmp指令，核心在于对26位立即数的处理上，26位立即数对应PC地址的2-27位，其余用0填充，实质上是左移了两位（？

![截屏2022-05-30 21.26.05](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202205302126560.png)

之后实现JR，与JAL搭配使用，

R1出来的数据返回到PC中，与程序的调用函数有关，调用函数前必须先存储函数起始点的位置，R1读取的数据即31位寄存器内的地址

![截屏2022-05-30 21.47.54](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202205302148170.png)

再之后是JAL，把当前的地址（PC已经加了4）写入寄存器中，1f即31号寄存器

![截屏2022-05-30 21.55.01](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202205302155395.png)

此时有红线很正常，因为没有设计控制信号

### 4. 补充

一开始设计的时候只考虑了基础的指令，而这个对应的是24条指令的电路，所以需要一些额外的设计

添加Syscall指令对应的电路

![截屏2022-06-01 21.14.00](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202206012114959.png)

对应的Syscall隧道，计算得到Halt停机指令和LedOut指令

![截屏2022-06-01 20.31.26](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202206012031417.png)

给PC添加停机

![截屏2022-06-01 20.32.10](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202206012032064.png)

产生Led的信号

![截屏2022-06-01 20.36.49](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202206012036587.png)

控制总周期数

![截屏2022-06-01 20.33.12](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202206012033122.png)

### 5. 完成

![202206012115373](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202206012115373.png)

## 三、控制器

根据实验给的文件“单周期硬布线控制器表达式自动生成”，通过运算器规格和控制信号产生条件填写真值表

1. ADD 加法
2. ADDI 立即数加
3. ADDIU 无符号数立即数加
4. ADDU 无符号数加
5. AND 与
6. ANDI 立即数与
7. SLL 逻辑左移
8. SRA 算术右移
9. SRL 算术右移
10. SUB 减
11. OR 或
12. ORI 立即数或
13. NOR 或非
14. LW 加载字
15. SW 储存字
16. BEQ 相等跳转
17. BNE 不相等跳转
18. SLT 小于置数
19. STI 小于立即数置数
20. SLTU 小于无符号数置数
21. J 无条件转移
22. JAL 转移并链接
23. JR 转移到指定寄存器中 If $v0==10 halt(停机指令)else 数码管显示$a0值
24. SYSCALL 系统调用 If $v0==10 halt(停机指令)else 数码管显示$a0值

![截屏2022-06-01 19.45.30](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202206011945300.png)

![截屏2022-06-01 19.45.43](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202206011945334.png)

得到真值表

![截屏2022-06-01 19.52.38](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202206011956626.png)

得到的逻辑表达式，复制到工程的分析组合逻辑电路里，就可以自动生成电路了

![截屏2022-06-01 19.56.49](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202206011958972.png)

将生成的两个组合起来，就完成了硬布线设计（已经连好线了



此时发现有蓝线的出现，说明还有问题存在，接下去就是找bug环节

![](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202206012112799.png)

## 四、测试

右击指令存储器编辑内容，打开.hex文件，之后进行测试