---
layout:     post
title:     OSI与TCP/IP协议簇、物理层
subtitle:  OSI与TCP/IP协议簇、物理层
date:       2020-5-8
author:     One-Punch-Man
header-img: "img/man-6.jpg"
catalog: true
tags: 
     - 网络安全
     - 基础知识
---

# OSI与TCP/IP协议簇

## 分层思想

- 将复杂的流程分解为几个功能相对单一的子过程
  - 整个流程更加清晰，复杂问题清晰化
  - 更容易发现问题并且可以针对性解决问题
- 使用网络的人未必需要知道数据的传输过程

## OSI七层模型

**开放式系统互联模型**（英语：**O**pen **S**ystem **I**nterconnection Model，缩写：OSI；简称为**OSI模型**）是一种概念模型，由国际标准化组织提出，一个试图使各种计算机在世界范围内互连为网络的标准框架。定义于ISO/IEC 7498-1。

- **应用层（Application Layer）**提供为应用软件而设的接口，以设置与另一应用软件之间的通信。例如：`HTTP、HTTPS、FTP、TELNET、SSH、SMTP、POP3、HTML`等。

- **表示层（Presentation Layer）**把数据转换为能与接收者的系统格式兼容并适合传输的格式。格式有`JPEG、ASCll、EBCDIC、加密格式`等 。

- **会话层（Session Layer）**负责在数据传输中设置和维护计算机网络中两台计算机之间的通信连接。

- **传输层（Transport Layer）**把传输表头（TH）加至数据以形成数据包。传输表头包含了所使用的协议等发送信息。例如:`TCP、UDP`。

- **网络层（Network Layer）**决定数据的路径选择和转寄，将网络表头（NH）加至数据包，以形成报文。网络表头包含了网络数据。例如`ICMP ARP IP（IPV4 IPV6）`。

- **数据链路层（Data Link Layer）**负责网络寻址、错误侦测和改错。当表头和表尾被加至数据包时，会形成[信息框](https://zh.wikipedia.org/wiki/資訊框)（Data Frame）。数据链表头（DLH）是包含了物理地址和错误侦测及改错的方法。数据链表尾（DLT）是一串指示数据包末端的字符串。例如以太网、无线局域网（Wi-Fi）和通用分组无线服务（GPRS）等。

  分为两个子层：逻辑链路控制（logical link control，LLC）子层和介质访问控制（Media access control，MAC）子层。

- **物理层（Physical Layer）**在局部局域网上传送[数据帧](https://zh.wikipedia.org/wiki/数据帧)（Data Frame），它负责管理电脑通信设备和网络媒体之间的互通。包括了针脚、电压、线缆规范、集线器、中继器、网卡、主机接口卡等。

![osi](..\img\day_11_01.png)

![osi](..\img\day_11_03.png)

## 数据的封装与解封 

![osi](..\img\day_11_02.png)

## TCP/IP协议簇

![osi](..\img\day_11_04.png)

## 设备与层的对应关系

![osi](..\img\day_11_05.png)

# 物理层

### 常用设备

- [光纤](https://zh.wikipedia.org/wiki/光纖)
- [CAT-5](https://zh.wikipedia.org/wiki/CAT-5)线
- [CAT-6](https://zh.wikipedia.org/wiki/CAT-6)线
- [CAT-7](https://zh.wikipedia.org/wiki/CAT-7)线
- [RJ-45](https://zh.wikipedia.org/wiki/RJ-45)接头
- [集线器](https://zh.wikipedia.org/wiki/集線器)
- [串口](https://zh.wikipedia.org/wiki/串口)
- [并口](https://zh.wikipedia.org/wiki/并口)

### 信号

#### 电信号

- **模拟信号**是指信息参数在给定范围内表现为连续的信号。 或在一段连续的时间间隔内，其代表信息的特征量可以在任意瞬间呈现为任意数值的信号，其信号的幅度，或频率，或[相位](https://baike.baidu.com/item/相位)随时间作连续变化，如广播的声音信号，或图像信号等。
- **数字信号**指幅度的取值是离散的，[幅值](https://baike.baidu.com/item/幅值)表示被限制在有限个数值之内。二进制码就是一种数字信号。二进制码受[噪声](https://baike.baidu.com/item/噪声)的影响小，易于有数字电路进行处理，所以得到了广泛的应用。

#### 光信号

#### 光纤类型

- 单模光纤：颜色一般为黄色
- 多模光纤：颜色一般为橙色/蓝色

#### 网线/双绞线

- 五类线
- 超五类线
- 六类线
- 七类线
- 超七类线

568A和568B是指用于8针[配线](https://baike.baidu.com/item/配线)（最常见的就是[RJ45](https://baike.baidu.com/item/RJ45)水晶头）模块插座/插头的两种颜色代码。按国际标准共有四种线序：T568A、T568B、USOC(8)、USOC(6)。一般常用的是前两种。

![网线](..\img\day_11_06.png)

![](..\img\day_11_07.jpg)