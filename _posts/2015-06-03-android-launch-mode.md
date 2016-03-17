---
layout: post
title: Android activity的四种launchMode
head: Android的四种launchMode是我们比较常用的基础点，但是有时候很容易就混淆了。
category: android 
tags: [launch mode]
---
{% include cooler/setup %}

Activity一共有四种启动方式：standard、singleTask、singleInstance、singleTop，四种方式各有个的特点，在不同情况下会使用不同的启动方式。
<br>
1. standard启动方式
这种方式是sdk种默认的方式，如果不给activity设置启动方式，那么就会默认的采用这种启动方式，这种方式在调用startActivity的时候，不管之前是否启动过，系统都会重新创建一个实例出来
<br>
<br>
2. singleTask启动方式
如果有其他task已经创建了这个activity，则会直接调用这个实例
<br>
<br>
3. singleInstance启动方式
新建一个task，并且该task中有且只有一个该activity的实例，如果后面再次调用startActivity，会重用这个实例
<br>
<br>
4. singleTop启动方式
如字面意思，在task栈顶只有一个实例，也就是如果当前栈顶是acticity A，如果启动intent又跳转到A，则不会产生新的实例，但如果A最初不在栈顶，则会产生一个实例（如：任务栈 A B C D，D在栈顶，这个时候有Intent启动了D，则启动后的任务栈情况为A B C D；如果有Intent启动了B，则任务栈的情况为： A B C D B）
<br>
<br>
以上为activity的四种方式，比较容易混淆，记录下来以备查看。