---
layout: post
title: linux shell报错bad interpreter: No such file or directory
head: 执行shell时遇到bad interpreter: No such file or directory。
category: tools
tags: [shell]
---
{% include cooler/setup %}

最近在linux使用ndk编译一些东西，就使用到了shell脚本，结果在执行的时候出现：
	
	bash: ./build_android.sh: /bin/bash^M: bad interpreter: No such file or directory

使用vim打开shell脚本，发现并没有问题，最后发现是由于shell脚本在windows下编写的，copy到linux后文本格式不对，所以只需要更改一下文本格式就可以了，用vim打开shell脚本，在命令行模式下：

	:set fileformat=unix

最后保存退出就可以执行了。