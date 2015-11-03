---
layout: post
title: Android打包App混淆
head: Android开发中有时候会需要混淆我们的app，不过这个真是个麻烦事，混淆过程中遇到各种各样的奇葩问题，这里记录一下在AndroidStudio中混淆App。
category: Android
tags: [proguard]
---
{% include cooler/setup %}

首先在app build.gradle中加入
{% highlight groovy %}
android {
	....

	buildTypes {
		release {
				minifyEnabled true
				proguardFiles 'proguard-rules.pro'
		}
	}

	....
}
{% endhighlight %}
其中`minifyEnabled`是用来开启或者关闭混淆的。
然后找到AndroidStudio下`proguard-rules.pro`文件，这个文件就是用来配置一些混淆所需要的规则。
先把一些基本规则放进去：
{% highlight text %}
## basic proguard rules

-keep public class * extends android.app.Application
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Fragment
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver

-keepattributes *Annotation*
-keepattributes Signature

-keepclasseswithmembernames class * {
	native <methods>;
}

-keepclasseswithmembernames class * {
	public <init>(android.content.Context, android.util.AttributeSet);
}

-keepclasseswithmembernames class * {
	public <init>(android.content.Context, android.util.AttributeSet, int);
}

-keep class * implements android.os.Parcelable {
	public static final android.os.Parcelable$Creator *;
}

-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

## support libraries
-dontwarn android.support.v4.**
-keep class android.support.v4.** { *; }
-keep interface android.support.v4.** { *; }
-keep public class android.support.v7.** { *; }
{% endhighlight %}
这样基本的规则就搞定了，可以来打包试一下，如果是新建工程那么是可以通过的，但是在正式的项目中往往还有其他东西，例如：三方类库、jni、aidl等等，这些如果不设置规则是不能编译通过的。
</br>
1.jni混淆问题
</br>
基本规则中有一条`-keepclasseswithmembernames class * { native <methods>; }`，可以排除native方法。
如果jni里面没有使用java类，这条可以忽略，但如果jni中使用了你自定义的类，那么就要注意了，比如定义了一个bean类，然后jni需要调用这个类，又或者jni需要调用java中的一个回调函数，这些情况下都需要把这些类排除在外。
</br>
2.aidl混淆问题
</br>
如果项目中使用了aidl，也需要注意，如果不排除就会导致在运行的时候出错（编译时并不会报错），需要将其排除，例如使用了`IPackageStatsObserver`，就需要`-keep class android.content.pm.IPackageStatsObserver { *; }`。
</br>
3.apache httpclient混淆问题
</br>
如果项目中使用了apache httpclient，然后sdk又升级到了api23，这时候编译就会报错找不到httpclient，原因是google在api23中将其全部移除了，这个时候需要在build.gradle中加入：
{% highlight groovy %}
android {
	....

	useLibrary 'org.apache.http.legacy'

	....
}
{% endhighlight %}
</br>
4.三方类库混淆问题
</br>
如果项目中使用了三方类库，一定要看作者是否提供了proguard，一般来说都会提供，直接加入到`proguard-rules.pro`文件就可以了。
</br>
</br>
ps:这里暂时记录这么多，混淆中还有很多问题，后面再慢慢添加上去。