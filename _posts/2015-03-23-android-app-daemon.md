---
layout: post
title: Android App Daemon
description: Android app process daemon, keep your app alive.
group: project
category: android
tags: [ndk, daemon]
---
{% include cooler/setup %}

项目地址：[Android App Daemon][1]

　　一直没空把App Daemon的原理整理一下，见不少人在问原理，我这里就把整个流程大概的说一下。
　　关于进程守护，从接触Android没多久就一直想实现，网上给出的方案也是各种各样，有双服务方式，有利用系统定时器方式的等等等等，但我都有过尝试，最终都没有达到自己想要的结果。后来想到自己以前在搞linux的时候用到子进程来处理一些任务，而Android正是基于linux的，觉得这样的话应该是可行的。最初直接在JNI的c代码中fork出一个子进程出来，然后在子进程中加一个while(1)，再在while中sleep并打开一个指定的service，这样一个最简单的守护完成了。但是后来测试发现，虽然能实现功能，不过使用adb shell查看进程(命令: ps | grep com.coolerfall....)，发现fork出来的进程的VSIZE(进程虚拟地址空间大小)和RSS(进程正在使用的物理内存的大小)都很大，而且UI线程有时候会出现莫名其妙的问题，于是进程守护也就暂时放下了。
　　后来由于项目需求，又不得不开始折腾进程守护。一次在看开源项目[afwall][2](android上的流量防火墙)，发现其中的命令是直接使用linux命令行的方式在执行的，这才想起linux可以直接编译一个可执行的二进制文件，然后在命令行中直接执行。看了看Android.mk的文档，加入`include $(BUILD_EXECUTABLE)`可以让c文件编译成在Android上运行的二进制文件，最后把以前的代码直接拿过来，一切OK了。
<br/>
<br/>
原理分析：
{% highlight c %}
int main(int argc, char *argv[])
{
	int i;
	pid_t pid;

	if ((pid = fork()) < 0)
	{
	    exit(EXIT_SUCCESS);
	}
	else if (pid == 0)
	{
		/* child process become session leader */
		setsid();
		/* change work directory */
		chdir("/");

		for (i = 0; i < 3; i ++)
		{
			close(i);
		}
		
		while(1)
		{
			sleep(interval);

			LOGD(LOG_TAG, "check the service once");

			/* start service */
			start_service(package_name, service_name);
		}
	}
	else
	{
		/* parent process, do nothing */
	}
}
{% endhighlight %}
由于要在shell中直接执行，因此这里使用`int main(int argc, char *argv[])`，让可执行文件有入口。frok出子进程之后，让子进程成为新的会话的领头进程，并与其父进程的会话组和进程组脱离，紧接着就是在子进程中定时去启动服务，这样一个简单的进程守护就OK了。
其中`start_service`为启动一个Service:
{% highlight c %}
/* start daemon service */
static void start_service(char *package_name, char *service_name)
{
	/* get the sdk version */
	int version = get_version();

	pid_t pid;

	if ((pid = fork()) < 0)
	{
		exit(EXIT_SUCCESS);
	}
	else if (pid == 0)
	{
		if (package_name == NULL || service_name == NULL)
		{
			LOGE(LOG_TAG, "package name or service name is null");
			return;
		}

		char *p_name = str_stitching(package_name, "/");
		char *s_name = str_stitching(p_name, service_name);
		LOGD(LOG_TAG, "service: %s", s_name);

		if (version >= 17 || version == 0)
		{
			execlp("am", "am", "start", "--user", "0",
					"-a", "android.intent.action.VIEW", "-d", url, (char *)NULL);
		}
		else
		{
			execlp("am", "am", "start", "-a", "android.intent.action.VIEW", "-d", url, (char *)NULL);
		}

		LOGD(LOG_TAG , "exit start-service child process");
		exit(EXIT_SUCCESS);
	}
	else
	{
		waitpid(pid, NULL, 0);
	}
}
{% endhighlight %}
这里说明下：execlp执行后，如果没有错误则不返回，有错时才有返回值，因此又fork了一个子进程，然后在子进程中启动指定的Service。其次是Android SDK版本小于17的执行命令有所不同，17及以上命令为：`execlp("am", "am", "startservice",
"--user", "0", "-n", s_name, (char *) NULL)`, 17以下:`execlp("am", "am", "startservice", "-n", s_name, (char *) NULL)`。获取version比较简单:
{% highlight c %}
#include <sys/system_properties.h>

/**
 * Get the version of current SDK.
 */
int get_version()
{
	char value[8] = "";
    __system_property_get("ro.build.version.sdk", value);

    return atoi(value);
}
{% endhighlight %}

由于编译出来的是可执行的二进制文件，所以调用就不像so库那样了。需要将可执行文件放在assets中，并在执行的时候将其copy至/data/data/<packagename>/app_bin文件夹下，然后在java中这样调用: 
{% highlight java %}
String cmd = "/data/data/<packagename>/app_bin/daemon"
Runtime.getRuntime().exec(cmd);
{% endhighlight %}

<br/>
进程守护大致的原理就是这样了，只要搞清楚了原理，其实是进程守护并不复杂。
<br/>
ps: 并不是所有手机都能用此方法实现进程守护，主要是因为现目前的进程清理软件不会清理c层fork出的进程，但有的手机（如小米），自带清理进程会清理掉应用相关的所有进程，故而不能实现进程守护。

[1]: https://github.com/Coolerfall/Android-AppDaemon
[2]: https://github.com/ukanth/afwall