---
layout: post
title: 使用MediaSessionCompat来实现Media Button的监听
head: "在API21之前，如果想实现线控，则只需要使用AudioManager.reregisterMediaButtonEventReceiver(ComponentName eventReceiver)即可，但是最近更新SDK之后发现这个API已经废弃掉了，推荐使用MediaSession来代替"
category: android
tags: [media]
---
{% include cooler/setup %}

API21之前的实现可以参考[这篇文章][1]，但是MediaSession是在新的api中加入的，我们可以使用android.support.v4.media.session.MediaSessionCompat：
{% highlight java %}
/**
 * Creates a new session.
 *
 * @param context The context.
 * @param tag A short name for debugging purposes.
 * @param mediaButtonEventReceiver The component name for your receiver.
 *            If null, this will attempt to find an appropriate
 *            {@link BroadcastReceiver} that handles
 *            {@link Intent#ACTION_MEDIA_BUTTON} from your manifest.
 *            A receiver is required to support platform versions earlier
 *            than {@link android.os.Build.VERSION_CODES#LOLLIPOP}.
 * @param mbrIntent The PendingIntent for your receiver component that
 *            handles media button events. This is optional and will be used
 *            on {@link android.os.Build.VERSION_CODES#JELLY_BEAN_MR2} and
 *            later instead of the component name.
 */
public MediaSessionCompat(Context context, String tag, ComponentName mediaButtonEventReceiver,
            PendingIntent mbrIntent) {
{% endhighlight %}
MediaSessionCompat的构造函数一共有四个参数，我们这里简单实现Media Button的监听，只需要context和mediaButtonEventReceiver即可，tag用于调试，mbrIntent设置为null即可。首先和API21之前一样实现了一个MediaButtonReceiver：
{% highlight java %}
public class MediaButtonReceiver extends BroadcastReceiver {
	@Override
	public void onReceive(Context context, Intent intent) {
		if (!Intent.ACTION_MEDIA_BUTTON.equals(intent.getAction())) {
			return;
		}

		KeyEvent event = intent.getParcelableExtra(Intent.EXTRA_KEY_EVENT);
		if (event == null || event.getAction() != KeyEvent.ACTION_UP) {
			return;
		}

		// do something
	}
}
{% endhighlight %}

接下来我们要做的就是new一个MediaSessionCompat：
{% highlight java %}
public class PlayerService extends Service {
	private MediaSessionCompat mMediaSession;
	
	@Override
	public void onCreate() {
		super.onCreate();

		ComponentName mbr = new ComponentName(getPackageName(), MediaButtonReceiver.class.getName());
		mMediaSession = new MediaSessionCompat(this, "mbr", mbr, null);
		/* set flags to handle media buttons */
		mMediaSession.setFlags(MediaSessionCompat.FLAG_HANDLES_MEDIA_BUTTONS);
		/* to make sure the media session is active */
		if (!mMediaSession.isActive()) {
			mMediaSession.setActive(true);
		}
	}

	@Override
	public void onDestroy() {
		mMediaSession.release();
	}
}
{% endhighlight %}
其他的和API21之前一样，需要在manifest中注册这个MediaButtonReceiver
{% highlight java %}
<receiver android:name="com.coolerfall.managers.receivers.MediaButtonReceiver">
    <intent-filter>
        <action android:name="android.intent.action.MEDIA_BUTTON"/>
    </intent-filter>
</receiver>
{% endhighlight %}
这样就可以实现和API21之前的监听效果了。

[1]: http://blog.csdn.net/qinjuning/article/details/6938436