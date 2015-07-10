---
layout: post
title: Android JNI注册的两种方式
head: Android JNI开发时，如何注册JNI的方法，Java才能调用，下面说一下JNI的两种注册方法。
category: android
tags: [jni]
---
{% include cooler/setup %}

第一种方法：静态注册
</br>
　　所谓静态注册就是调用java的命令工具javah来生成头文件，然后再实现头文件中的所有函数即可。这种方法比较简单，首先在命令行中（我这里使用的是windows cmd，linux、mac是一样的），进入到src目录下，然后执行：

	$ javah -d E:\SourceCode\Android\JniTest\ -jni com.coolerfall.HelloJni

其中-d表示生成的头文件的输出目录，可以自行设置，com.coolerfall.HelloJni是包含有native方法的类，native方法如：

	public static native void init();

最后生成一个com_coolerfall_player_HelloJni.h头文件，接下来就可以新建一个c文件实现这些函数就ok了。
</br>
</br>
第二种方法：动态注册
</br>
　　静态方法虽然用起来方便，只需要使用一句命令行就搞定了，但是这种方法我们不知道jni的注册过程是怎样的，而且如果新添加一个方法后，又得重新生成一次，比较麻烦，动态注册就可以避免这个问题。和静态注册的区别在于，不使用javah，而由我们自己来写注册函数等等。
</br>
　　我们可以新建一个c文件，比如init.c，然后在里面添加
{% highlight c %}
#include <jni.h>

static JNINativeMethod g_methods[] = {
	{"init", "()V", (void *)init},
	{"start", "(Ljava/lang/String;JJ)V", (void *)start},
};

int register_native_methods(JNIEnv* env, const char* class_name,
		JNINativeMethod* methods, int num_methods)
{
	jclass clazz;
	clazz = (*env)->FindClass(env, class_name);
	if (clazz == NULL) {
		return JNI_FALSE;
	}
	if ((*env)->RegisterNatives(env, clazz, methods, num_methods) < 0) {
		return JNI_FALSE;
	}

	return JNI_TRUE;
}

jint JNI_OnLoad(JavaVM *vm, void *reserved)
{
	JNIEnv* env = NULL;

	if ((*vm)->GetEnv(vm, (void **) &env, JNI_VERSION_1_4) != JNI_OK)
	{
		return 0;
	}

	if (!register_native_methods(env, "com/coolerfall/HelloJni", g_methods,
			sizeof(g_methods) / sizeof(g_methods[0])))
	{
		return 0;
	}

	return JNI_VERSION_1_4;
}
{% endhighlight %}
Java层调用System.loadLibrary("xxx")的时候，会首先进入JNI_OnLoad这个函数里面，因此，我们就在这里面调用register_native_methods对JNI的一些列方法进行注册，最终在register_native_methods调用了jni函数RegisterNatives来对native方法注册到对应的类上去，这样就完成了jni的注册，java就可以调用jni的方法了。使用这种方法时，添加一个native方法就非常方便了，直接在g_methods数组里面添加新的方法即可。
</br>
　　关于g_methods数组里面方法的签名规则可以查看[Android JNI类型、方法签名规范][1]。


[1]: http://coolerfall.com/android/jni-signature/