---
title: Android Studio更新时连接服务器失败问题解决
category: tools
tags: [android studio]
---

最近被墙得厉害，连AS都不能自动更新了，最终找到了个比较简单有效的办法。
<!-- more -->

解决方法很简单，打开AS安装目录下的bin目录，找到&nbsp;`studio64.exe.vmoptions`&nbsp;（32位找&nbsp;`studio.exe.vmoptions`&nbsp;即可）这个文件，往里面添加几行：
```text
-Djava.net.preferIPv4Stack=true

-Didea.updates.url=http://dl.google.com/android/studio/patches/updates.xml  
-Didea.patches.url=http://dl.google.com/android/studio/patches/
```
保存退出，重启AS，不出意外就能更新了。