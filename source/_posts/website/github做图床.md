---
title: github做图床
date: 2021-07-13
author: fzxhub
cover: true
img: 
summary: 使用github仓库做图床，开源jsdelivr的cdn加速，效果不错。👍
categories: website
tags:
  - Hexo
  - github
  - 图床
---

## github建立仓库
1. 新建仓库（仓库一定公开）

## 上传图片到该仓库
1. 上传的图片可以存储到根目录，也可以建立文件，但访问的时候要注意即可

## 访问
1. 使用开源jsdelivr的cdn进行访问，可以加快速度。

```
https://cdn.jsdelivr.net/gh/{github用户名}/{github仓库名}@{仓库分支名}/文件名
```
> 注意：如果仓库还有文件夹，文件名中也好包含文件夹路径