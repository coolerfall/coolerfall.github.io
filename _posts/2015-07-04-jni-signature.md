---
layout: post
title: Android JNI类型、方法签名规范
head: Android JNI签名规范容易记混，记录以备查看。
category: android
tags: [jni]
---
{% include cooler/setup %}

在Android开发中不免会使用到JNI，JNI编程中可以使用javah等工具自动生成jni的头文件，但是如果想自己手动注册，那么就需要了解JNI方法的签名规范：

<table class="table table-bordered table-striped table-condensed">
	<th>Java类型</th>
	<th>JNI签名</th>
	<tr>
		<td>boolean</td>
		<td>Z</td>
	</tr>
	<tr>
		<td>byte</td>
		<td>B</td>
	</tr>
	<tr>
		<td>char</td>
		<td>C</td>
	</tr>
	<tr>
		<td>short</td>
		<td>S</td>
	</tr>
	<tr>
		<td>int</td>
		<td>I</td>
	</tr>
	<tr>
		<td>long</td>
		<td>J</td>
	</tr>
	<tr>
		<td>float</td>
		<td>F</td>
	</tr>
	<tr>
		<td>double</td>
		<td>D</td>
	</tr>
	<tr>
		<td>Java类(例: String)</td>
		<td>L全类名;(例: Ljava/lang/String; )</td>
	</tr>
	<tr>
		<td>数组(例: int[])</td>
		<td>[基本类型签名(例: [I )</td>
	</tr>
	<tr>
		<td>Java方法(例: start(String path, long pos, long duration))</td>
		<td>(参数签名...)返回值签名(例: (Ljava/lang/String;JJ)V)</td>
	</tr>
</table>
要注意的是java类的签名，最后的分号不要忘记。java方法签名中，括号里面是所有参数的类型签名，中间无间隔，括号外面是返回值类型签名。