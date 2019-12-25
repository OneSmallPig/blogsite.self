---
title: 关于IDEA创建项目的一些朦胧
date: 2019-12-25 14:21:57
categories: IDEA
tags: 
	- use
	- IDEA
---

IDEA创建Spring项目时往往会生成一些文件，这些文件有时候用不上但是又不知道与项目有什么样的关系，粗略了解这些文件的作用，可能对缓解一些强迫症状会有奇效？：）

<!-- more -->

### 1.project.xml文件

IDEA中的.iml文件是项目标识文件，缺少了这个文件，IDEA就无法识别项目。

### 2.HELP.md文件

项目的帮助文档。

### 3.mvnw

linux上处理meaven版本兼容问题的脚本。

### 4.mvnw.cmd文件

windows上处理mevan版本兼容问题的脚本。

### 5..gitignore文件

git版本控制插件模板，用于控制文件和文件夹不被git提交。（不用git做版本控制时可删除）