---
title: hexo使用说明
date: 2021-07-13
author: fzxhub
cover: true
img: 
summary: Hexo建站使用说明，记录我最常使用的指令防止忘记怎么使用。
categories: website
tags:
  - Hexo
---

## 前提
安装 Hexo 相当简单，只需要先安装下列应用程序即可：
1. Node.js (Node.js 版本需不低于 10.13，建议使用 Node.js 12.0 及以上版本)
	- Windows：通过 nvs（推荐）或者 nvm 安装。
	- Mac：使用 Homebrew 或 MacPorts 安装。
	- Linux（DEB/RPM-based）：从 NodeSource 安装。
	- 其它：使用相应的软件包管理器进行安装，可以参考由 Node.js 提供的 指导。
2. Git
	- Windows：下载并安装 git.
	- Mac：使用 Homebrew, MacPorts 或者下载 安装程序。
	- Linux (Ubuntu, Debian)：sudo apt-get install git-core
	- Linux (Fedora, Red Hat, CentOS)：sudo yum install git-core

## 安装hexo

所有必备的应用程序安装完成后，即可使用 npm 安装 Hexo。

```bash
$ npm install -g hexo-cli 
```

## 建站

1. 安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。

```bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```

2. 新建完成后，指定文件夹的目录如下：

```bash
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

3. 本地预览

```bash
$ hexo server
```

4. 推到远端

```bash
$ hexo clean && hexo deploy
```

推到远端前提配置

```yaml
deploy:
  type: git
  repo: 仓库地址
  branch: 仓库分支
```
