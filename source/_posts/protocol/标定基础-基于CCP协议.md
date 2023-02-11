---
title: 标定基础-基于CCP协议
date: 2022-08-28 16:00:00
author: fzxhub
cover: true
img: /image/asap/theory.png
summary: 介绍ASAP相关协议，适用于汽车控制器的标定、调试底层实现。
categories: protocol
tags:
  - CCP
  - 标定
  - 汽车控制器
---

## 背景
标定协议是汽车控制在编写程序后，部分功能实现的部分参数是需要在实车上才能确定的，当然在实车上调试过程中也需要监测一些数据才能进行调试工作。

在一些需要调试较少的系统中，可以一边调试一边优化参数重新下载程序。对于汽车这样复杂系统就变得不可能。因此诞生一种标定系统。程序工程师将程序编写架构搭建好，在编程过程中需要预知那些变量需要标定(实时修改)，那些变量需要观测(实时查看)。系统程序搭建好后，标定工程师即可在实车上实时查看、调试控制器。因此诞生一种标定协议。


## 标定协议的底层实现

汽车控制器程序本质运行在SOC上，为了性能和稳定性是无GUI的。因此需要将标定工程师手中的PC作为标定的输入和输出。标定中需要实时和友好的标定软件(电脑端软件)通信。在汽车中广泛存在CAN控制器域网。自然首要选择它作为标定的通信通道。

汽车控制器的SOC使用C语言。C代码中的参数值可以在RAM(初始值本质在ROM)中，或者ROM中。我们需要标定的参数如果当变量保存在RAM中，则修改后掉电就会恢复为初始值；如果存储在ROM，则不能对其进行修改。

因此大佬们想了一个办法，标定的参数在SOC上存储两份，RAM、ROM中各一份。当汽车控制器上电是将ROM标定区的数据搬移到RAM标定区，程序运行时使用RAM标定区的参数，标定过程中就能实时修改RAM标定参数，当标定参数确定以后，将RAM标定区数据拷贝到ROM标定区，这样下次上电则是最新的标定参数。

![原理](/image/asap/theory.png)

当然仅仅只支持标定软件监测的参数(观测量)直接是定义在RAM中的变量，用来实时存储程序运行的部分参数并支持发送给标定软件即可，也不关心存放在ROM的初始值。

- 注意：本节主要描述标定底层原理，实际工程系统会复杂一些。

## 标定协议规范

### ASAP标准
是几家德国汽车制造商联手一些著名的汽车电子设备制造商于1991年成立了ASAP标准组织， ASAP的英文全称是The working group for thestandardization of applicationsystems(应用系统标准化工作小组)，它的目标是使在汽车电子设备研发过程中相关的测试，标定，诊断方法及工具能够兼容并互换。

ASAP3是应用系统，即测试，标定，诊断系统(MCD Measurement， Calibration， Diagnosis System)到自动化系统的接口规范。这里的自动化系统可以是一个测量仪器的指示装置或汽车的燃油测量装置等。

ASAP2又称为ASAP描述文件，是控制单元内部数据描述文件的规范。 ASAP2文件用来具体描述电子控制单元内部的数据信息，包括数据存储的规范，数字量到物理量的转换规范等。

ASAP1是控制单元到MCD系统的接口规范，ASAP1规范又细分为ASAP1b与ASAP1a。ASAP1b接口下包括一个符合ASAP标准的驱动程序，硬件接口及电子控制单元。因此ASAP1b接口规范保证了MCD与ECU之间的通信，不受所选通信媒介及不同ECU供应商的限制。其中ASAP1a是到ECU端的数据通信的物理及逻辑接口规范，包括通过CAN总线对ECU进行标定的协议规范。

### ASAM标准组织及其规范
1998年ASAM小组成立，其英文全称是Association for Standardization of Automation and Measuring System(自动化及测量系统标准化小组)。ASAM标准是ASAP标准的扩展和衍生，在新的ASAM标准中，ASAP标准 变名为ASAM MCD(ASAM Measurement, Calibration and Diagnosis)，原来的ASAP1、ASAP2、ASPA3规范在新的标准下分别为ASAM-MCD 1MC、ASAMMCD 2MC、ASAM-MCD 3MC。

![架构](/image/asap/total.png)

## CCP协议命令格式

CCP协议是运行在CAN接口之上的标定协议(ASAP1a)，CCP的全称是CAN Calibration Protocol。 CCP协议遵从CAN2.0B通信规范，支持11位标准与29位扩展标识符。数据都是8字节对齐。

![CRODTO](/image/asap/crodto.png)

CRO是主设备(标定软件)发送给从设备(控制器)的指令；DTO是从设备发送给主设备的数据。

- CRO中CMD(命令码)范围0x00-0xFF，具体含义见指令详解。
- DTO中PID(包识别码)范围0x00-0xFF，具体含义见DTO详解。
- CRO、DTO中的CRT(命令计数码)在一问一答通讯模式中每次加一。对应CRO、DTO的计数码相等。

### DTO详解

![DTO](/image/asap/dto.png)

DTO的PID分为三类：
1. CRM是PID=0xFF的情况，主要是在一问一答通讯模式中回复CRO。
2. EM是PID=0xFE的情况，主要是从设备主动发送数据给主设备，例如从设备遇到一些错误等消息需要上报。
3. DAQ是PID=0x00-0xFD的情况，主要是数据采集模式下从设备主动发送采集数据到主设备。

### DAQ详解

DAQ就是从设备向标定软件发送的数据采集信息。也就是将当前控制器中参数发送给标定软件。

![DAQ](/image/asap/daq.png)

- 一个ODT表最多有7Byte的数据，一个ODT就对应一个DTO，也就是一帧CAN报文。
- 一个DAQ表有多个ODT表。每张DAQ表不可能只有7Byte的数据量，需要多个ODT来传输。
- 一个标定系统有多张DAQ表。每张DAQ表的采集周期一般不一样。

为什么这样区分呢？因为标定系统需要的参数可能需要监测周期不一样。例如一些数据需要10ms报告一次，一些数据需要100ms报告一次。这样利于标定工程师观测参数。因此每张DAQ表的采集周期不一样。

### DAQ举例

- 一个标定系统有两张DAQ
    - DAQ#0采集周期为10ms，ODT编号0x00-0x3F。
        - 采集数据Data1(uint8) = 0x01
        - 采集数据Data2(uint32) = 0x00000002
        - 采集数据Data3(uint32) = 0x00000003
    - DAQ#1采集周期为100ms，ODT编号0x40-0x7F。
        - 采集数据Data4(uint16) = 0x0004
        - 采集数据Data5(uint32) = 0x00000005

![DAQ](/image/asap/daqexp.png)

## CCP指令详解

### 必须支持指令

#### 0x01: CONNECT(M)

![CONNECT](/image/asap/cmd01.png)

站地址是一个16bit的数字。

#### 0x17: EXCHANGE_ID(M)

![EXCHANGE_ID](/image/asap/cmd17.png)

可用资源掩码: 当bit=TRUE，指定的资源或函数可用。

保护资源掩码: 当bit=TRUE，指定的资源或功能被保护防止未经授权的访问(需要解锁)。

ID用于设置MTA0为上传请求的地址，方便随后使用UPLOAD命令。

#### 0x02: SET_MTA(M)

![SET_MTA](/image/asap/cmd02.png)

设置MTA的地址，地址=32bit地址+扩展。

MTA0用于DOWNLOAD、UPLOAD、DNLOAD_6、SELECT_CAL_PAGE、CLEAR_MEMORY、PROGRAM、PROGRAM_6、MOVE。

MTA1用于MOVE。

#### 0x03: DOWNLOAD(M)

![DOWNLOAD](/image/asap/cmd03.png)

包含的指定长度(大小)的数据块将被复制到设备的内存，从当前内存传输地址0(MTA0)开始。MTA0指针将会后加Size值。

#### 0x04: UPLOAD(M)

![UPLOAD](/image/asap/cmd04.png)

包含的指定长度(大小)的数据块将被上传到标定软件，从当前内存传输地址0(MTA0)开始。MTA0指针将会后加Size值。

#### 0x14: GET_DAQ_SIZE(M)

![GET_DAQ_SIZE](/image/asap/cmd14.png)

返回指定的DAQ列表的大小作为可用对象的数量并清除当前列表。如果指定的列表号不是Size=0应该返回。

#### 0x15: SET_DAQ_PTR(M)

![SET_DAQ_PTR](/image/asap/cmd15.png)

初始化DAQ列表指针，以便后续写入DAQ列表.

#### 0x16: WRITE_DAQ(M)

![WRITE_DAQ](/image/asap/cmd16.png)

将一个条目(单个DAQ元素的描述)写入由DAQ列表定义的DAQ列表指针(见SET_DAQ_PTR)。定义了以下DAQ元素的大小:1byte，2bytes (type word)，4字节(type long / Float)。ECU可能不支持每个元素和2或4字节的单独地址扩展元素的大小。兼容ECU的限制是主控设备的责任。限制可以在从设备描述文件(ASAP2)中定义。

#### 0x06: START_STOP(M)

![START_STOP](/image/asap/cmd06.png)

该命令用于启动或停止数据采集或准备同步数据开始指定的DAQ列表。LastODTNum编号指定哪些ODT(从0到此DAQ列表的最后ODT号码)应传送。事件通道号指定有效地确定数据传输时序的通用信号源。

#### 0x07: DISCONNECT(M)

![DISCONNECT](/image/asap/cmd07.png)

断开备用设备的连接。断开可以是暂时的，设置从设备处于“脱机”状态或参数0x01终止校准会话。终止会话将使所有状态信息失效，并重置从保护的地位。临时断开不会停止DAQ消息的传输。

#### 0x1B: GET_CCP_VER(M)

![GET_CCP_VER](/image/asap/cmd1B.png)

此命令用于对协议版本进行相互识别主设备和从设备在通用协议版本上达成一致。这命令的执行时间应该在EXCHANGE_ID命令之前。


### 可选支持指令

#### 0x12: GET_SEED(O)

![GET_SEED](/image/asap/cmd12.png)

用于解锁设备时获取种子。

#### 0x13: UNLOCK(O)

![UNLOCK](/image/asap/cmd13.png)

发送生成的密钥解锁设备。

#### 0x23: DOWNLOAD6(O)

![DOWNLOAD6](/image/asap/cmd23.png)

包含的固定长度(6字节)的数据将被复制到设备的内存，从当前内存传输地址0(MTA0)开始。MTA0指针将会后加6。

#### 0x0F: SHORT_UP(O)

![SHORT_UP](/image/asap/cmd0F.png)

指定长度(大小)的数据块，从源地址开始将被复制到对应的DTO数据字段。MTA0指针保持不变。

#### 0x11: SELECT_CAL_PAGE(O)

![SELECT_CAL_PAGE](/image/asap/cmd11.png)

该命令的功能依赖于ECU的实现。之前初始MTA0指向选定为当前活动的校准数据页的起点通过这个命令分页。

#### 0x0C: SET_S_STATUS(O)

![SET_S_STATUS](/image/asap/cmd0C.png)

设置从节点了解校准会话的当前状态。

#### 0x0D: GET_S_STATUS(O)

![GET_S_STATUS](/image/asap/cmd0D.png)

获取从节点了解校准会话的当前状态。

#### 0x0E: BUILD_CHKSUM(O)

![BUILD_CHKSUM](/image/asap/cmd0E.png)

返回由MTA0定义的内存块的校验和结果(内存传输区域起始地址)和块大小。校验和算法可能是制造商和/或项目具体，它不是本规范的一部分。

#### 0x10: CLEAR_MEMORY(O)

![CLEAR_MEMORY](/image/asap/cmd10.png)

此命令可用于在重编程前擦除FLASH EPROM。的MTA0指针指向要擦除的内存位置。

#### 0x18: PROGRAM(O)

![PROGRAM](/image/asap/cmd18.png)

指定长度(大小)的数据块将被编程成非易失性存储器(FLASH, EEPROM)，从当前MTA0开始。MTA0指针将会后加Size值。

#### 0x22: PROGRAM6(O)

![PROGRAM6](/image/asap/cmd22.png)

国定长度(6字节)的数据块将被编程成非易失性存储器(FLASH, EEPROM)，从当前MTA0开始。MTA0指针将会后加Size值。

#### 0x19: MOVE(O)

![MOVE](/image/asap/cmd19.png)

指定长度(大小)的数据块将从MTA 0定义的地址复制(源指针)指向MTA 1定义的地址(目的指针)。

#### 0x20: DIAG_SERVICE(O)

![DIAG_SERVICE](/image/asap/cmd20.png)

从设备执行请求的服务，并自动设置内存传输地址MTA0到CCP主设备(主机)可能来自的位置随后上传所请求的诊断服务返回信息。

#### 0x21: ACTION_SERVICE(O)

![ACTION_SERVICE](/image/asap/cmd21.png)

从设备执行请求的服务，并自动设置内存传输地址MTA0到CCP主设备的位置随后上传所请求的操作服务返回信息(如果适用)。

#### 0x05: TEST(O)

![TEST](/image/asap/cmd05.png)

该命令用于测试指定站点地址的从站是否存在可供CCP沟通。此命令不建立逻辑连接，也不会在指定的从站中触发任何活动。站地址(Station)指定为小端字节顺序的数字(Intel格式)。

#### 0x08: START_STOP_ALL(O)

![START_STOP_ALL](/image/asap/cmd08.png)

该命令用于启动所有DAQ列表的周期传输使用之前发送START_STOP命令(启动/停止方式= 2)以同步方式作为“准备开始”。该命令用于停止所有DAQ列表(包括未启动的)的定期传输同步。

#### 0x09: GET_ACTIVE_CAL(O)

![GET_ACTIVE_CAL](/image/asap/cmd09.png)

控件中当前活动的校准页的起始地址从设备。


### Command Return Codes

![Command Return Codes](/image/asap/returncode.png)

## ASAP2文件规则

该部分后续完善...
