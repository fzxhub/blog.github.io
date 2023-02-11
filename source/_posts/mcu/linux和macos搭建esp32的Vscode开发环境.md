---
title: linux和macos搭建esp32的Vscode开发环境
date: 2021-09-28
author: fzxhub
cover: true
img: /image/esp32/anzhuang.jpg
summary: linux和macos在VScode中搭建esp32的开发环境。
categories: mcu
tags:
  - esp32
  - 嵌入式
---

## 前提说明

> 说明：第一步到第五步是安装测试ESP- IDF的步骤，和我上一篇文章同理。大家有其他方法完成也是可以的，如果之前已经测试ESP- IDF，直接去第六步
> 注意：直接去第六步的前提是ESP- IDF目录在～/esp下，工具安装在～/.espressif下，如果自己有改动这些文件夹，需改根据Vscode的ESP- IDF插件的引导设置。

## 背景

esp32开发方式有许多
1. 如在Arduino软件中Arduino框架开发方式，网上有许多教程
2. 在platformIO中进行Arduino或者idf方式开发，platformIO封装不错
3. 使用官方idf在linux、macos、windows开发

本次记录在linux和macos搭建esp32的Vscode开发环境。

[乐鑫官方文档](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32/get-started/index.html#get-started-get-prerequisites)

[乐鑫ESP-IDF VS Code 插件快速操作指南(Windws)](https://www.bilibili.com/video/BV17p4y167uN?p=1)


## 第一步：基础工具安装

在linux或者macos中搭建esp32的环境，先要安装一些基础工具，如python等等。macos可以使用Homebrew、macports安装相关的工具以及依赖。如果部分工具已经安装过可以直接跳过。
- linux

``` shell
apt purge vim-common
apt install vim
apt install git
apt install git wget flex bison gperf python3 python3-pip 
apt install python3-setuptools cmake ninja-build ccache 
apt install libffi-dev libssl-dev dfu-util libusb-1.0-0
```
- macos

``` shell
brew install vim
```

## 第二步：克隆esp-idf仓库

esp-idf项目是分子模块进行团队开发的SDK。因此直接克隆的仓库不能直接使用需要将子模块也克隆才能使用，我在这里卡了许久。遇到问题，使用多注意命令行的提示，对应想办法解决。

- linux、macos

``` shell
#方案一：这是直接递归克隆乐鑫在GitHub上的esp-idf仓库，但是国内容易失败，可以试试
git clone --recursive  https://github.com/espressif/esp-idf.git
#方案二：先克隆esp-idf仓库，然后拉取子模块
git clone https://github.com/espressif/esp-idf.git
git submodule update --init --recursive
#方案三：1.这是使用国内gitee托管克隆仓库
git clone https://gitee.com/EspressifSystems/esp-idf.git
#方案三：2.克隆国内gitee的工具仓库
git clone https://gitee.com/EspressifSystems/esp-gitee-tools.git
#方案三：3.复制esp-gitee-tools中的submodule-update.sh到esp-idf，然后执行submodule-update.sh
./submodule-update.sh
```
备注：
1. 这一步主要思想就是将就是将官方的esp-idf仓库完整克隆下来，在这个过程中不能报错，大家也可以查看该仓库中的.gitmodules中看看有哪些子模块。在这些模块在该项目的位置在GitHub中是以超链接的方式链接的。只克隆esp-idf到本地时，这些文件夹是空的，我们只要想办法将这些空文件夹中的内容填进去就可以，各种办法都可以。
2. 官方文档是引导我们在～/文件夹建立一个esp文件夹，然后将esp-idf放入其中，建议大家都这样做。

## 第三步：安装编译器、调试器等工具

esp-idf包有了，需要安装esp32的编译器、调试器等等工具，其实些工具就是gcc、openocd等工具，这些工具主要是官方根据esp32修改过的定制版本。在esp-idf中，官方为我们写好下载脚本。我们只要执行脚本文件就可以安装了。esp32有esp32、esp32s2、esp32s3等等系列。我们可选系列安装或者全安装。这里我全安装

- linux、macos

``` shell
#设置从乐鑫官方下载方式
export IDF_GITHUB_ASSETS="dl.espressif.com/github_assets"
#执行安装脚本
./install.sh
```

备注：
1. 当看到提示“You can now run：. .export.sh”,表示安装成功。
2. 该步主要下载编译编译器、调试器等等工具，是下载工具包然后本地解压安装。如果依然失败，可以设置从乐鑫官方下载的方式，或者直接想办法下载安装包到指定目录等等方式尝试。

## 第四步：设置环境变量

此时，您刚刚安装的工具尚未添加至 PATH 环境变量，无法通过“命令窗口”使用这些工具。因此，必须设置一些环境变量，这可以通过 ESP-IDF 提供的另一个脚本完成。

- linux、macos

``` shell
#设置环境变量
. .export.sh
#如果当前不在esp-idf文件夹中
. ~/esp/esp-idf/export.sh
```

备注：
1. 当终端关闭后设置环境变量失效。需要再次运行export.sh设置。
2. 可以使用方法设置成打开终端自动设置环境变量，自行查找方案。

## 第五步：编译、烧录、验证

复制esp-idf/examples/get-started/下的hello_world工程或者其他工程到自己的工作目录进行编译、烧录、验证工作。

``` shell
#到工作目录下
cd ~/esp/hello_world
#设置目标芯片
idf.py set-target esp32
#打开配置界面
idf.py menuconfig
#编译
idf.py build
#烧录
idf.py -p PORT [-b BAUD] flash
#打开串口监视
idf.py -p PORT monitor
```

## 第六步：在Vscode中安装ESP- IDF插件

这步没有什么要注意的，直接在商店安装即可

![安装](/image/esp32/anzhuang.jpg)

## 第七步：配置ESP- IDF插件

1. 查看/命令面板/Congfigure ESP-IDF，就会打开一个配置界面
2. 配置界面有三个选项：快速、可配置、使用现有；如果没有前面1-6步的操作，可以直接使用快速按钮，会下载所有使用资源及工具，ESP-IDF会从GitHub下载。我的会失败。老老实实使用前面的方法。

![配置](/image/esp32/peizhi.jpg)

> 注意：macos下安装了ESP- IDF插件后，进入配置界面没有三个配置选项，可能是插件BUG，我的隔两天再打开的时候就有配置选项了

## 第八步：编译、烧录、验证

和第五步一样，我们复制一个实例工程，或者打开插件的实例工程编译、下载、测试即可。
![实例](/image/esp32/shili.jpg)
![按键](/image/esp32/anjian.jpg)

> 注意：linux编译通过，但是烧录会失败，会弹出提示，访问USB权限不够，原因是openocd访问USB失败，根据linux下Vcsode的提示复制60-openocd.rules到/etc/udev/rules.d/60-openocd.rules，依然不可行，我们在该文件中添加一行：KERNEL=="ttyUSB[0-9]*", MODE="0666"  然后重新插入USB就可以了