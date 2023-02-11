---
title: 瑞萨芯片在CS+中分区方法
date: 2022-12-20 18:00:00
author: fzxhub
cover: true
img: /image/rscs/section.png
summary: 瑞萨芯片在CS+开发需要添加分区时，先规划好分区的地址在section修改
categories: mcu
tags:
  - 瑞萨
  - CS+
---

## 瑞萨芯片在CS+中分区方法


CS+的分区是以图形配置。需要添加分区时，先规划好分区的地址，然后来section来添加。

### 进入分区

![进入分区设置](/image/rscs/section.png)

### 添加分区

注意添加分区时先添加地址栏，然后在新建地址栏后面建立标签。

![增加分区设置](/image/rscs/add.png)

### 添加分区名称

填写自己分区名字后需要在后面添加分区类型：.bss .data .const .text

- .bss：表示该分区存储的变量没有初始值，一般直接使用给RAM的分区后缀。
- .data：表示该分区存储的变量有初始值，如果该区在RAM，会没有初始值。
- .test：表示该分区存储代码段。一般用于ROM。

![分区类型](/image/rscs/name.png)

### 代码中使用分区

如下代码，使用分区CAL，则编译器会查找CAL.data段来存储该变量。

```c
#pragma section r0_disp32 "CAL"
char test[8] = "test";
#pragma section
```

### 分区映射

一般.data有RAM和ROM。ROM用于存储初始值。RAM是代码寻址使用的地址，如果只有RAM则该变量没有初始值，只有ROM则该变量不能修改。地址映射，则是将ROM存储的变量的地址映射到RAM区。

![分区地址映射](/image/rscs/map.png)

### 上电数据迁移

地址映射完成但是该RAM区上电是没有初值的，需要代码进行搬移。

![分区初始化](/image/rscs/datainit.png)

```asm
	; when the section has the initial value
	.section    ".INIT_DSEC_PE0.const", const
	.align      4
	.dw		   #__s.data,  #__e.data,  #__s.data.R
	.dw         #__s.PE0.data,  #__e.PE0.data,  #__s.PE0.data.R
	.dw         #__sCAL.data,  #__eCAL.data,  #__sCAL_RAM.data
	
	; when the section without initial value
	.section    ".INIT_BSEC_PE0.const", const
	.align      4
	.dw         #__s.bss,   #__e.bss
	.dw         #__s.PE0.bss,   #__e.PE0.bss
	;.dw	   #__s.R_RFD_BSS.bss, #__e.R_RFD_BSS.bss
```

注意：

1. .INIT_DSEC_PE0.const分区是放的需要数据搬移的起始结束地址，.INIT_BSEC_PE0.const分区是放的需要数据初始化为0的起始结束地址。该文件后面会有具体数据搬移和初始化为0的代码。
2. 需要分区搬移数据的分区标号可以在map文件中查看
