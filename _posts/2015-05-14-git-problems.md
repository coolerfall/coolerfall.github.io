---
layout: post
title: 记录git使用中遇到的一些小问题
head: 现在在项目中git的使用已经比较频繁了，难免会遇到各种各样的小问题，这里记录下这些问题。
category: tools
tags: [git]
---
{% include cooler/setup %}

<br>
1.在push大文件的时候遇到

	error: RPC failed; result=22, HTTP code = 411
	fatal: The remote end hung up unexpectedly
	fatal: The remote end hung up unexpectedly
这个是因为http buffer不够造成的，可以简单配置git来解决：
	
	git config http.postBuffer 67108864
2.想要把已经push的文件（夹）加入到.ignore中
<br>
直接添加到.ignore中是不能成功的，因为已经push到服务器了，需要先删除本地缓存的文件：

	git rm -r --cached .idea
然后再将其添加到.ignore中，最后git push就ok了