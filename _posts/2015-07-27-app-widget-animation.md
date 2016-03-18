---
layout: post
title: Android AppWidget中实现动画效果
head: 最近在写AppWidget的时候，想实现一个淡入的动画效果，由于AppWidget只支持几种view并且没有提供实现动画的方法，折腾了很久发现只有使用LayoutAnimation可以勉强实现动画效果。
category: android
tags: [animation]
---
{% include cooler/setup %}

<br>
这里就拿淡入动画作为例子。首先在`res/anim`中新建一个动画`fade_in.xml`：
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<alpha xmlns:android="http://schemas.android.com/apk/res/android"
       android:duration="1200"
       android:fromAlpha="0.0"
       android:interpolator="@android:anim/accelerate_interpolator"
       android:toAlpha="0.8"/>
{% endhighlight %}
然后再新建一个layoutAnimation，`widget_fade_in.xml`：
{% highlight xml %}
<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
                 android:animation="@anim/fade_in"/>
{% endhighlight %}
动画效果准备好后，接下来就是在AppWidget布局中使用了。比如AppWidget的布局，`widget_layout.xml`如下：
{% highlight xml %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/widget_bg">

    <RelativeLayout
        android:id="@+id/widget_layout_iv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layoutAnimation="@anim/widget_fade_in">

        <ImageView
            android:id="@+id/widget_iv"
            android:layout_width="100.0dip"
            android:layout_height="100.0dip"
            android:layout_centerInParent="true"/>

    </RelativeLayout>

    <TextView
        android:id="@+id/widget_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/app_name">
    </TextView>

</LinearLayout>
{% endhighlight %}
要让ImageView实现淡入动画，需要将其单独加入到一个layout中，然后在layout中加入`android:layoutAnimation="@anim/widget_fade_in"`。这样ImageView所在的layout就有动画效果了，不过这个动画效果只会在AppWidget第一次加载的时候有，如果想每隔一段时间去切换，就需要让AppWidget不断的实现重新加载的过程，需要使用到RemoteViews的removeAllViews和addView方法，这两个方法可以让AppWidget的layout刷新来实现重复动画的效果。再新建一个layout文件`widget_image.xml`:
{% highlight xml %}
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/widget_layout_iv"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center"
    android:layoutAnimation="@anim/widget_fade_in">

    <ImageView
        android:id="@+id/widget_iv"
        android:layout_width="100.0dip"
        android:layout_height="100.0dip"
        android:layout_centerInParent="true"/>

</RelativeLayout>
{% endhighlight %}
这里面的内容保持和`widget_layout.xml`中要实现动画的layout一致就行了。
<br>
最后需要在AppWidgetProvider中使用removeAllViews和addView方法：
{% highlight java %}
public class ExampleWidget extends AppWidgetProvider {
	@Override
	public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
		updateWidget(context, appWidgetManager, appWidgetIds);
	}

	public static void updateWidget(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
		String pkgName = context.getPackageName();
		RemoteViews views = new RemoteViews(pkgName, R.layout.widget_layout);
		RemoteViews subView = new RemoteViews(pkgName, R.layout.widget_image);
		views.removeAllViews(R.id.widget_layout);
		views.addView(R.id.widget_layout, subView);
	}
}
{% endhighlight %}
最后就是在Activity或者Service中，在需要更新的时候调用`updateWidget`就有动画效果了。