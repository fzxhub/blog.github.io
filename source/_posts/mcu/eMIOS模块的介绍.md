---
title: eMIOS模块的介绍
date: 2022-05-09 12:00:00
author: fzxhub
cover: true
img: /image/emios/CB.png
summary: MCU常用外设eMIOS的简单原理介绍，辅助更好使用该外设。
categories: mcu
tags:
  - eMIOS
  - 单片机
  - 外设
---

## 背景

eMIOS(Enhanced Modular IO Subsystem)增强型模块化IO子系统，在多种控制器的外设模块中存在，本次来了解eMIOS的原理。助力开发中更好使用它。
本次介绍主要资料来源NXP S32K3的相关手册。该手册介绍的比较详细，更加方便我们了解eMIOS的原理，原理会了，在其他MCU中使用大同小异。

## eMIOS模块说明
### 组成说明

每个eMIOS模块都有24或者更多个通道（UC），这些通道都是相互独立的且又是互相配合，但是这些通道在结构上并不是一模一样，基本以8个通道为一组，多组通道组成一个eMIOS模块。具体差异可以参考手册。这里主要原理说明。

![模块](/image/emios/BD.png)

### 时钟源、计数器说明

每个eMIOS的每个通道都有自己的计数源（Counter Bus或者叫CNT）也叫该通道的内部计数器，在eMIOS模块的时钟源驱动下进行计数。

部分的eMIOS通道可以将自己内部的计数器分享出去共给其他通道使用，这样就可以做到计数器同步来做一些高级的应用。在实际使用时，根据应用、功能需求等选择使用哪个计数器作为该通道的计数器来工作。例如产生单个PWM使用内部计数器即可；产生步进电机的驱动脉冲则使用Counter Bus B/C/D/E可完成多个通道波形产生来控制可解决波形同步问题。

![模块](/image/emios/CB.png)

例如S32K3的eMIOS中，Counter Bus A是一个全局的计数器，它是由CH23产生分享的，可以给任何通道使用；Counter Bus B是一个局部的计数器，它是由CH0产生分享的，可以给1-7任何通道使用；Counter Bus C是一个局部的计数器，它是由CH8产生分享的，可以给9-15任何通道使用；Counter Bus D是一个局部的计数器，它是由CH16产生分享的，可以给17-22任何通道使用；Counter Bus F同理Counter Bus A，是由CH22分享产生的。

![通道](/image/emios/UC.png)

例如S32K3的eMIOS的一个通道的内部框图如上。计数器来源可以来源Counter Bus A、Counter Bus B/C/D/E、Counter Bus F。

## eMIOS SAIC模式

Single Action Input Capture  Mode 就是输入捕获，检测到一个上升沿或者下降沿，UC就生成一个Flag信号，同时捕捉当前Counter Bus的值到AS2。也可以上升沿和下降沿同时采集。

![SAIC](/image/emios/SAIC.png)
![SAICB](/image/emios/SAICB.png)

## eMIOS SAOC模式

Single Action Output Capture  Mode就是输出匹配模式。给AS2写入一个值后，当Counter Bus的值与AS2相等时，这个时候就会产生一个Flag信号，同时控制输出跳变或者翻转。

![SAOC](/image/emios/SAOC.png)
![SAOCT](/image/emios/SAOCT.png)

## eMIOS IPWM模式

Input Pulse Width Measurement Mode，这个模式就是用来测量两个连续不同沿之间的宽度，即测量一个电平宽度。当检测到第一个沿，捕捉Counter Bus的值存入AS2，当检测到第二个沿，再捕捉Counter Bus的值存入BS2并产生一个Flag信号。第一个边沿为上升沿第二个边沿为下降沿表示测量高电平宽度；第一个边沿为下升沿第二个边沿为上降沿表示测量低电平宽度。

![IPWM](/image/emios/IPWM.png)

## eMIOS IPM模式

Input Period Measurement Mode，这个模式用来测量两个连续相同沿的宽度，和 IPWM类似。测量的是信号周期，即检测到第一个沿（上升沿或者下降沿），捕捉Counter Bus的值存入AS2，当检测到第二个沿（与第一个沿相同），再捕捉Counter Bus的值存入BS2并产生一个Flag信号。

![IPM](/image/emios/IPM.png)

## eMIOS DAOC模式

Double Action Output Compare Mode，这个模式相比较SAOC，它有两个比较输出，就是当第一个匹配事件发生时，将输出信号翻转，第二个匹配事件发生时，将输出信号再次翻转。第一个事件匹配成功后可生产Flag信号也可以不生成Flag信号（可设置），而第二个匹配成功后则产生Flag信号。

![DAOC](/image/emios/DAOC.png)

## eMIOS PEC模式

Pulse Edge Counting Mode，可测量信号的脉冲数或者边沿数。外部信号输入检测到有效信号后计数器累加，并且在当计数器CNT值发生匹配事件AS1后，复位计数器CNT并启动计数，发生匹配事件BS1后，停止计数并且输出Flag信号。

![PEC](/image/emios/PEC.png)

## 更多模式介绍后续完善...