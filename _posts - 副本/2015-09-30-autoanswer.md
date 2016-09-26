---
layout: post
title: bat脚本自动应答
tags: bat 
categories: 脚本
---

<div class="toc"></div>

<br/>
----------
**自动应答一个参数**

可以使用以下脚本命令：

    @echo 参数 | 执行语句

----------
**自动应答多个参数**

在自动应答多个参数的时候可以使用文件重定向，先将需要的参数写入到一个文件中，每个参数间回车相隔，当执行完命令后，再将文件删除掉：

    echo 123>k.txt
	echo 456>>k.txt
	echo 789>>k.txt
	type k.txt | 执行语句
	del k.txt

   

