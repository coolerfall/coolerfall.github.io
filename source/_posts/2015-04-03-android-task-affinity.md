---
title: Android taskAffinity与桌面快捷方式
category: android
tags: [taskAffinity]
---

最近在做一个像豌豆荚游戏文件类似的东西，但是遇到个奇怪的问题，就是应用打开后home键退出，然后再点击文件夹，会同时弹出文件夹以及home退出之前的界面，经过一番折腾，算是整明白了问题。首先说说taskAffinity，每个application创建的时候，都会有taskAffinity，默认情况下同一个application下的所有activity都属于同一个taskAffinity，都会在相同的task中。而这里创建的文件夹快捷方式，相当于是让其指定到了应用的某个acitivity，如果没有给这个activity指定taskAffinity的话，那么将会与前面的activity运行在相同的task中，也就是在打开快捷方式后，若前面的activity未finish掉，这个时候会从栈中弹出，出现我所遇到的问题。最后在文件夹快捷方式指定的activity中加入`android:taskAffinity=":icon"`，问题就解决了。