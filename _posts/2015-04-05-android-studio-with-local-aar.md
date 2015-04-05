---
layout: post
title: Android Sutdio使用本地aar包
head: Android Studio使用gradle构建项带来了很大的方便，可以只用在app目录下的build.gradle中的dependencies加入一句代码就搞定了。
category: android 
tags: [android studio]
---
{% include cooler/setup %}

有时候由于网络的原因，可能导致无法下载jcenter或者maven center的库，我们需要使用本地包。本地jar包放入lib目录下，可以使用：
{% highlight groovy %}
compile fileTree(dir: 'libs', include: ['*.jar'])
{% endhighlight %}
aar包就稍微麻烦一点，首先在app下的build.gradle文件中加入：
{% highlight groovy %}
repositories{
	flatDir{
		dirs 'libs'
	}
}
{% endhighlight %}
然后将下载好的aar包放入libs目录下，然后再在dependencies中加入依赖，比如我这里使用了一个material-dialogs：
{% highlight groovy %}
dependencies {
	compile 'com.afollestad:material-dialogs:0.6.7.2@aar'
}
{% endhighlight %}
这样就可以在代码中使用aar包了。