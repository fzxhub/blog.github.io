---
title: BootLoader开发介绍
date: 2022-04-19 12:00:00
author: fzxhub
cover: true
img: /image/boot/First.png
summary: 单片机BootLoader开发介绍，并以NXP的S32系列的一款MCU举例说明。
categories: mcu
tags:
  - BootLoader
  - 单片机
---


## 该文档结构说明

1. BootLoader的介绍
2. 介绍BootLoader的整体功能与模块（通信、NVM驱动、存储管理、跳转管理模块）
3. 以NXP的S32系列的一款MCU举例BootLoader的开发

## BootLoader介绍

BootLoader就是驻留MCU非易失性存储器中的一段程序加载代码，每次复位后，都会运行Boot。它会检查是否有来自通信总线的远程程序加载请求，如果有则进入BootLoader模式，建立与程序下载端(通常为PC上位机)的总线通信并接收通信总线下载的应用程序、解析其地址和数据代码，运行NVM（None Valitale Momory-非易失性存储器）驱动程序，将其编程到NVM（一般为Flash）中，并校验其完整性，从而完成应用程序更新。如果没有来自通信总线的远程程序加载请求，则直接跳转到应用程序复位入口函数（复位中断ISR），运行APP（应用程序）。

![RUN](/image/boot/First.png)

本文介绍的BootLoader属于MCU层面的BootLoader，不同与U-Boot那样强大的功能，BootLoader的主要功能：

1. 通信模块：与远程程序下载端建立可靠的总线通信以获取要更新应用程序；
2. NVM驱动模块：NVM驱动将应用程序的代码和数据编程到NVM中并校验；
3. 存储管理：需要根据MCU的RAM和ROM大小、Boot和应用的需求来进行存储的合理规划；
4. 跳转管理：根据跳转条件跳转到应用程序处执行应用代码，或者从应用代码跳回Boot进行升级操作。

## 通信模块

通信模块主要作用是建立程序下载端与MCU端的通信联系通道，可以进行握手、数据发送、数据接收等。

1. 总线的选择：总线通信可选择UART、SPI、USB、CAN、LIN、以太网。只具体需要用到某种通信总线取决于实际应用，在汽车领域一般选择CAN。
2. 总线驱动：在bootloader中必须开发相应的通信总线外设驱动程序，实现基本的数据发送和接收功能。
3. 通信的可靠性：为了保证通信的可靠性，必须开发一个基于通信总线完善的通信协议，应用程序下载端和bootloader之间需要建立请求命令、确认、等待、错误重传、数据打包、数据解包等机制。bootloader根据不同的请求命令完成不同的任务并确认操作是否完成(ACK)以及数据是否正被确完整的传输，若出现数据错误(通过校验和或者ECC实现)，需要进行自动重传。汽车领域一般需要实现CAN_UDS或者CAN_CPP协议栈来处理通信。
4. 通信下载端：通过在PC上开发GUI软件，实现要求的总线通信协议，一般在其底层都是通过调用相应的总线设备，如USB转CAN/LIN的转发器设备的动态库(DLL)的API接口来实现数据的收发，相应的总线USB转发设备都会提供相应的驱动库(DLL)。因此bootloader开发者一般还需具备一定的PC上位机软件开发能力。

## NVM驱动模块

NVM（None Valitale Momory-非易失性存储器）一般包括其MCU片内集成的用于存放数据的EEPROM或者Data-Flash和用于存储程序代码/数据的Code-Flash/Program-Flash以及MPU扩展的片外NOR Flash或者NAND-Flash；NVM驱动程序包括对NVM的擦除(erase)、编程(program)和校验(verify)等基本操作，也包括对NVM的加密(secure)/解密(unsecure)和加保护(protection)/解保护(unprotection)操作。

1. 工作速度：由于NVM的工作速度一般较CPU内核频率和总线频率低，所以运行NVM驱动前必须对NVM进行初始化，将设置分频器其工作频率设置为正常工作所需频率范围。
2. 总线竞争：MCU片内的NVM同一个block上不能运行NVM的驱动对其自身进行擦除和编程操作，否则会传出read while write的总线访问冲突（每个NVM block只有一条数据总线，一个时刻只能进行读出或者写入，不支持同时读出和写入）。另一个方案是可以将NVM驱动拷贝到MCU的RAM中运行。
3. 驱动存储位置：NVM的驱动程序驻留在NVM中，如果出现堆栈溢出等意外程序跑飞意外运行NVM驱动程序则会造成NVM内容意外擦除丢失或者修改的情况。因此需要对关键数据或代码（比如bootloader本身）进行保护以防止意外修改，或者更为安全的方法是不将NVM驱动程序存放在NVM中，而是在bootloader最开始通过上位机将其下载到RAM中运行，bootloader结束后将该区域RAM清除，从而避免由于意外运行NVM驱动程序造成的NVM数据丢失和修改。

## 内存管理

### MCU执行程序原理

MCU内部闪存(FLASH)地址有一个起始地址，一般情况下，程序文件就从此地址开始写入。程序启动后，将首先从“中断向量表”取出复位中断向量执行复位中断程序完成启动，而这张“中断向量表”的起始地址是FLASH的起始地址加4字节处，也是复位中断向量的地址。当中断来临，MCU内部硬件机制亦会自动将 PC指针定位到“中断向量表”处，并根据中断源取出对应的中断向量执行中断服务程序。

![RUN](/image/boot/RUN1.jpeg)

如图，MCU复位后，先从 0X08000004 地址取出复位中断向量的地址，并跳转到复位中断服务序，如图标号1所示，在复位中断服务程序执行完之后，会跳转到我们 的 main 函数，如图标号2所示，而我们的 main 函数一般都是一个死循环，在 main 函数执行过 程中，如果收到中断请求(发生重中断)，此时 CPU强制将PC指针指回中断向量表处，如图标号3所示;然后，根据中断源进入相应的中断服务程序，如图标号4所示;在执行完中 断服务程序以后，程序再次返回 main 函数执行，如图标号5所示。

### ROM、RAM的空间划分

![ROM](/image/boot/ROM.jpg)

Boot和应用程序的链接文件中，对NVM的地址空间分配必须分开独立，不能重叠(overlap)，但其RAM分配没有约束，两者都可以使用整个RAM空间，因为跳转到应用工程后，将启动代码将重新初始化RAM；

根据升级需求，我们需要将ROM分区处理，一部分存放Boot代码，一部分存放APP的代码。APP存放方案有两种，方案一是APP只有一份在ROM中，升级前需要将APP区域擦除后才能写入新的APP代码。方案二是APP有两份在ROM中，当前有效的APP不用擦除就可以下载新的APP代码。

#### 方案一优缺点
1. ROM空间利用率高；
2. 如果新的APP升级失败，则不能启动APP，只能留在Boot中。

#### 方案二优缺点
1. ROM空间利用率低；
2. 如果新的APP升级失败，可以留在旧的APP中，不影响之前的功能。

## 跳转管理

bootloader和应用程序分别是两个完整的MCU软件工程，各自都由自己的启动代码、main()函数、链接文件、外设驱动程序和中断向量表。

### Boot到应用程序的跳转方法
开发使用bootloader后，每次MCU复位之后都将首先运行Boot，若符合运行APP的条件则直接跳转到应用程序复位函数地址。找到应用程序复位函数地址的方案有两种：
1. 通过链接文件固定应用程序的复位启动函数地址
2. 从应用程序中断向量表的复位向量地址获取

### Boot跳转到APP

Boot更新完应用程序并校验其完整性OK之后，将用到的外设(比如CAN/LIN通信总线模块、定时器、GPIO等)寄存器恢复到复位后的默认状态，然后根据APP的复位函数地址跳转。

### APP跳转到Boot

在APP运行时需要跳回Boot时，可以使用软件复位即可回到Boot。

## S32为例BootLoader开发

### RAM、Flash的需求分析以及分区规划

1. 找出MCU的内存分布图/表，找到Flash、RAM的起始地址、结束地址。

![S32Memory](/image/boot/S32Memory.png)

2. 根据需求画出Flash、RAM的分区逻辑图（S32的Flash最小擦除单位4K）

![S32Memmap](/image/boot/S32Memmap.jpg)

### Boot项目RAM、Flash分区

1. Boot项目链接脚本的修改，划出我们需要的分区

```c
MEMORY
{
  /* Flash ALL = 128K*/
  m_interrupts          (RX)  : ORIGIN = 0x00000000, LENGTH = 0x00000400  //1K
  m_flash_config        (RX)  : ORIGIN = 0x00000400, LENGTH = 0x00000010  //16Byte
  m_text                (RX)  : ORIGIN = 0x00000410, LENGTH = 0x0001FBF0  //~=127K
  /* SRAM_L ALL = 64K*/    
  share_noinit          (RW)  : ORIGIN = 0x1FFF0000, LENGTH = 0x00000400 //Add sections 1K
  flashDrv              (RW)  : ORIGIN = 0x1FFF0400, LENGTH = 0x00000400 //Add sections 1K
  m_data                (RW)  : ORIGIN = 0x1FFF0800, LENGTH = 0x000F8000 //62K
  /* SRAM_U ALL = 60K*/
  m_data_2              (RW)  : ORIGIN = 0x20000000, LENGTH = 0x0000F000 //60K
}

SECTIONS
{
    .share_noinit (NOLOAD) :
    {
      . = ALIGN(4);
      KEEP(*BootMange.o(.share_noinit))
      KEEP(*(.share_noinit))
    } > share_noinit
}

```

2. Startup文件修改，增加了新的分区，在复位的时候需要进行清除操作来同步ECC的校验数据，防止ECC的报错。但是share_noinit段只能在硬件复位情况下才进行清除，因为share_noinit我们的用途是用来做BOOT和APP之间的交互使用。在APP跳到BOOT的时候（软件复位）不能清除share_noinit段。

```c
/* Clear the flash driver data section */
while(flashDrv_end != flashDrv_start)
{
    *flashDrv_start = 0;
    flashDrv_start++;
}
/* Clear Noinit section when Power-On Reset in order to initialize the ECC detection hardware */
if (McuIsPowerOnReset())
{
    while(share_noinit_end != share_noinit_start)
    {
        *share_noinit_start = 0;
        share_noinit_start++;
    }
}
```
### APP项目RAM、Flash分区

1. APP项目链接脚本的修改，划出我们需要的分区，share_noinit段因为和Boot共享所以必须同地址同长度。

```c
MEMORY
{
  /* Flash  ALL = 896K*/
  CAL				   (RX)  : ORIGIN = 0x00020000, LENGTH = 0x00001000  //Add sections 4K
  m_interrupts          (RX)  : ORIGIN = 0x00021000, LENGTH = 0x00000400 //1K
  m_flash_config        (RX)  : ORIGIN = 0x00021400, LENGTH = 0x00000010 //16byte
  m_text                (RX)  : ORIGIN = 0x00021410, LENGTH = 0x000DEBF0 //~=891K
  /* SRAM_L ALL = 64K*/
  share_noinit          (RW)  : ORIGIN = 0x1FFF0000, LENGTH = 0x00000400 //Add sections 1K
  m_data                (RW)  : ORIGIN = 0x1FFF0400, LENGTH = 0x0000FC00 //63K 
  /* SRAM_U ALL = 60K*/
  m_data_2              (RW)  : ORIGIN = 0x20000000, LENGTH = 0x0000F000 //60K
}

SECTIONS
{
    .share_noinit (NOLOAD) :
    {
      . = ALIGN(4);
      KEEP(*BootMange.o(.share_noinit))
      KEEP(*(.share_noinit))
    } > share_noinit
    
    . = ORIGIN(CAL);
    .cal :
    {
        KEEP(*(.cal))
    } > CAL
}

```
### 跳转处理

1. 在从APP跳到BOOT时需要做Stay in Boot的标志作用，预先分区了share_noinit段，因此我们在跳转处理时进行标志的变量需要放到share_noinit段。

```c
typedef struct
{
  uint8 JumpType;
  uint8 JumpType_Lv;
} TypeOfJump_t;

//定义方法1：
__attribute__((section(".share_noinit"))) TypeOfJump_t ;

//定义方法2：
#pragma section ".share_noinit"
TypeOfJump_t TypeOfJump;
#pragma section
```

2. 根据链接脚本定义，我们知道了APP的分区开始，就知道了APP的复位中断函数的地址，将这个地址强转化为函数指针后，跳到该地址执行该函数，就跳到了APP的代码执行了。

```c
void BOOT_CheckIsNeedJump(void)
{
	static uint32 JumpAddress;
	uint8 temp = ~TypeOfJump.JumpType_Lv;

	if (TypeOfJump.JumpType == temp)
	{
		switch(TypeOfJump.JumpType)
		{
			case BOOT_JUMP_APP:
				Boot_SetTypeOfJump(NO_NEED_JUMP);
				DISABLE_INTERRUPTS();
				JumpAddress = *(uint32 *)(FLASH_APP_START_ADDRESS + 4);
				((JumpFunction_t)JumpAddress)();
				break;
			case APP_JUMP_BOOT:
				Boot_SetTypeOfJump(NO_NEED_JUMP);
				BOOT_StayInBoot();
				DS_SetReprogPattenValid();
				break;
			case BOOT_JUMP_EOL:
				Boot_SetTypeOfJump(NO_NEED_JUMP);
				DISABLE_INTERRUPTS();
				JumpAddress = *(uint32 *)(FLASH_EOL_START_ADDRESS + 4);
				((JumpFunction_t)JumpAddress)();
				break;
			default:
				Boot_SetTypeOfJump(NO_NEED_JUMP);
				break;
		}
	}
	else
	{
		/* Power On */
		Boot_SetTypeOfJump(NO_NEED_JUMP);
	}
}

```

### Boot与FlashDriver对接

1. FlashDriver项目中来编写FlashDriver的实体，为了Boot能调用我们的FlashDriver的实体，我们需要为我们实现的功能建立索引结构体，FlashDriver项目的结构体和Boot的结构体保持一一对应，在Boot中就能调用FlashDriver的功能。

- FlashDriver

```c
typedef struct
{
	char FingerPrint[8];
	FlashFun_t flashInitFun;
	FlashFun_t flashEraseFun;
	FlashFun_t flashWriteFun;
} FlashHeader_t;

typedef struct
{
	unsigned char errorCode;
	unsigned int  address;
	unsigned int  length;
	unsigned char *data;
	flash_ssd_config_t* flashSSDConfig;
} FlashParam_t;

typedef void (* FlashFun_t)( FlashParam_t *flashParam );

//FlashDriver项目中索引结构体，放在FlashDriver分区的头部方便在Boot中索引
__attribute__((section(".flashDrvHeader"))) FlashHeader_t FlashHeader =
{
	FLASH_FINGER_PRINT,
	&FLASHDRV_Init,
	&FLASHDRV_Erase,
	&FLASHDRV_Write
};
```

- Boot

```c
typedef struct
{
	unsigned char errorCode;
	unsigned int  address;
	unsigned int  length;
	unsigned char *data;
	flash_ssd_config_t* flashSSDConfig;
} FlashParam_t;

typedef void (* FlashFun_t)( FlashParam_t *flashParam );

typedef struct
{
	char FingerPrint[8];
	FlashFun_t flashInitFun;
	FlashFun_t flashEraseFun;
	FlashFun_t flashWriteFun;
} FlashHeader_t;

//索引结构体起始地址就是FLASHDRV_START_ADDRESS，从该地址按FlashHeader_t检索则得到FlashDriver的函数

#define FLASHDRV_ADDRESS                (FLASHDRV_START_ADDRESS)
#define FLASH_DRIVER_VALID()            (memcmp(((FlashHeader_t*)FLASHDRV_ADDRESS)->FingerPrint, FLASH_FINGER_PRINT, sizeof(((FlashHeader_t*)FLASHDRV_ADDRESS)->FingerPrint)) == 0)
#define FLASH_DRIVER_INIT(flashParam)   (((FlashHeader_t*)FLASHDRV_ADDRESS)->flashInitFun)(flashParam)
#define FLASH_DRIVER_ERASE(flashParam)  (((FlashHeader_t*)FLASHDRV_ADDRESS)->flashEraseFun)(flashParam)
#define FLASH_DRIVER_WRITE(flashParam)  (((FlashHeader_t*)FLASHDRV_ADDRESS)->flashWriteFun)(flashParam)


```
