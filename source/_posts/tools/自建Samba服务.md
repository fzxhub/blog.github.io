---
title: 自建Samba服务
date: 2022-07-17
author: fzxhub
cover: true
img: 
summary: 自建Samba服务，用于局域网内Windows、Mac、Linux、IOS文件分享。
categories: tools
tags:
  - Samba
  - 工具
---

## 背景

在现在的家庭局域网中，已经有多台联网设备了，设备之间可能需要传输一些文件共享，大部分情况我们都是使用微信等聊天工具进行共享。操作不太方便。因此有了自建一个轻量的网盘服务，在家庭中使用。
了解了一下，我们的Mac、Windows、Linux都支持Samba服务。而且Mac、Windows不需要任何额外的软件支持，加上老旧电脑闲置，不再购买NAS设备。而且Samba操作还是很简单的。

## 安装Ubuntu

这一步将我旧电脑安装成Linux设备。Linux稳定好用。可以安装桌面版和服务器版。桌面版有图形操作、服务器版没有图形操作，根据自己爱好安装。也可以安装其他Linux操作系统。当然也可以Mac、Windows，可以自己折腾。
这里我使用Ubuntu服务器版，以下教程使用Ubuntu。安装步骤按照网上做，教程多。

## 安装配置Samba服务

1. 安装samba服务器。

```shell
sudo apt-get install samba samba-common
```

2. 创建一个共享文件夹，用于共享使用。这里我们需要给读写执行权限，防止后面在这个文件夹下不能读写执行的操作。

```shell
mkdir /home/fzxhub/share
sudo chmod 777 /home/fzxhub/share
```

3. 添加samba用户及密码，后面连接该服务需要提供用户名和密码。该指令后会提示输入密码即可。

```shell
sudo smbpasswd -a 输入需要增加的用户名
```

4. 修改samba的配置文件，该配置文件配置samba的工作模式等等。


```shell
sudo vi /etc/samba/smb.conf
```

```shell
[share]
comment = share folder
browseable = yes
path = /home/fzxhub/share   #你共享文件夹路径
create mask = 0700
directory mask = 0700
valid users = fzxhub     
force user = fzxhub
force group = fzxhub
public = yes
available = yes
writable = yes
```

5. 重启Samba服务器

```shell
sudo service smbd restart
```

6. 查询服务器端的IP地址，该IP地址后续连接需要，建议静态IP，防止该服务器的IP地址变更后不能访问问题。可以在Ubuntu上设置为静态IP，也可以在我们的路由器上将IP地址和MAC地址绑定后来固定IP地址。

## 其他局域网设备连接Samba服务

### Mac连接Samba服务

在Mac的网络中即可发现服务器的名称，点击名称后，再点击连接身份输入，输入我们在Samba服务设置的用户名和密码后就连接好了。

### IOS连接Samba服务

在IOS设备的文件APP中，IOS设备Apple官方的APP。右上角选择连接服务器输入"smb://IP地址"，选择注册用户，然后输入用户名和密码即可连接。

### Windows连接Samba服务

在Windows的运行或者资源管理器的路径栏输入"\\\IP地址"，即可访问，然后在文件夹上右键选择映射网络驱动器即可加入盘符访问，如需要输入用户名和密码输入即可。

