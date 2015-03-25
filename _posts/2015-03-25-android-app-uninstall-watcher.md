---
layout: post
title: Android App Uninstall Watcher
description: App uninstall watcher, used to monitor the uninstall of your app.
group: project
category: android
tags: [ndk, uninstall]
---
{% include cooler/setup %}

　　很多应用在卸载后都会弹出一个网页做用户卸载反馈，这就需要监听App的卸载，但是应用一旦卸载就不会再执行任何程序了，如何才能弹出网页，答案就是在应用开启时就fork出一个子进程来，在进程中对App进行卸载监听。
　　在linux中有个东西叫inotify，可以对指定的文件进行监听（包括修改，删除等等），基本的流程就是inotify_init->inotify_add_watch->inotify_event，在inotify_event读取操作的时候是阻塞的，一直会等到指定的文件变动后才会往下执行:
{% highlight c %}
int main(int argc, char *argv[])
{
	char *app_dir = "/data/data/com.xx.yy";
	char *watch_file_path = str_stitching(app_dir, "/uninstall.watch");

	pid_t pid = fork();
	if (pid < 0)
	{
		return;
	}
	else if (pid == 0)
	{
		/* inotify init */
		int fd = inotify_init();
		if (fd < 0)
		{
			exit(EXIT_FAILURE);
		}

		int w_fd = open(watch_file_path, O_RDWR | O_CREAT | O_TRUNC,
				S_IRWXU | S_IRWXG | S_IRWXO);
		if (w_fd < 0)
		{
			exit(EXIT_FAILURE);
		}

		close(w_fd);

		/* add watch in inotify */
		int watch_fd = inotify_add_watch(fd, watch_file_path, IN_DELETE);
		if (watch_fd < 0)
		{
			exit(EXIT_FAILURE);
		}

		void *p_buf = malloc(sizeof(struct inotify_event));
		if (p_buf == NULL)
		{
			LOGD(LOG_TAG, "malloc inotify event failed");
			exit(EXIT_FAILURE);
		}

		/* read will block process */
		read(fd, p_buf, sizeof(struct inotify_event));

		free(p_buf);
		inotify_rm_watch(fd, IN_DELETE);

		open_browser(url);
	}
}
{% endhighlight %}

与[App Daemon][1]一样，使用命令行的方式来调用wathcer，所有这里也`int main(int argc, char *argv[])`，最终将这个c文件编译成可执行文件。为了要监听某个文件的删除，首先在/data/data/<packagename>/下新建了一个uninstall.watch文件，在后面我们将对此文件进行监听，然后同样的，为了不妨碍主进程，这里fork出一个子进程，在子进程里面进行操作。接下来是关键的地方：调用`inotify_init()`初始化inotify，接着将刚刚新建的uninstall.watch文件加入`inotify_add_watch(fd, watch_file_path, IN_DELETE)`中监听，这里是监听该文件的删除（第三个参数还可以是IN_CREATE, IN_MODIFY, IN_MOVED_TO等等，根据需求更改），接下来就是读取`read(fd, p_buf, sizeof(struct inotify_event))`，这个函数会一直阻塞直到条件满足才会继续往下执行，最后释放并加入打开浏览器。打开浏览器也比较简单了:
{% highlight c %}
/* open browser with specified url */
void open_browser(char *url)
{
	/* the url cannot be null */
	if (url == NULL || strlen(url) < 4) {
		return;
	}

	/* get the sdk version */
	char value[8] = "";
	__system_property_get("ro.build.version.sdk", value);

	int version = atoi(value);
	/* is the version is greater than 17 */
	if (version >= 17 || version == 0)
	{
		execlp("am", "am", "start", "--user", "0", "-n",
				"com.android.browser/com.android.browser.BrowserActivity",
				"-a", "android.intent.action.VIEW",
				"-d", url, (char *)NULL);
	}
	else
	{
		execlp("am", "am", "start", "-n",
				"com.android.browser/com.android.browser.BrowserActivity",
				"-a", "android.intent.action.VIEW",
				"-d", url, (char *)NULL);
	}
}
{% endhighlight %}
和[App Daemon][1]类似，也是使用execlp来调用am命令打开默认浏览器，最后就会弹出一个指定的网页了。
　　以上是卸载监听的基本流程，但我在实际操作过程中遇到了比如调试应用，实际上是覆盖，但这时候也会弹出网页等等，所有在子进程中的操作有所改动和优化，并在打开浏览器前使用了[libcurl][2]请求服务器，以满足不需要打开网页的需求，具体请查看[wathcer.c][3]。
</br>
</br>
ps:此方法和[App Daemon][1]一样由于系统原因不能适配所有手机，请注意。

[1]: http://coolerfall.com/android/android-app-daemon
[2]: http://curl.haxx.se/libcurl/
[3]: https://github.com/Coolerfall/Android-AppUninstallWatcher/blob/master/app/src/main/jni/watcher/watcher.c