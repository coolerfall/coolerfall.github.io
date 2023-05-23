---
layout: post
title: "Error: The output jar is empty. Did you specify the proper '-keep' options"
category: android
tags: [proguard]
---

最近在写个小工具混淆Android lib包，用到了proguard命令行的方式，结果出现`Error: The output jar is empty. Did you specify the proper '-keep' options`的错误。"出现这个错误是因为我这里混淆的lib包里面所有方法都没有调用，所以输入类全被压缩移除，导致没有输出。解决这个问题很简单，只需要加上`-dontshrink`就可以了。