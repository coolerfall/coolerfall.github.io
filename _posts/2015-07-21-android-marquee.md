---
layout: post
title: TextView实现跑马灯效果
head: 跑马灯效果是TextView自带的一个属性，使用TextView来实现单个、多个跑马灯效果比较简单。
category: android
tags: [widget]
---
{% include cooler/setup %}

</br>
1.单个跑马灯效果
这种比较简单，只需要在布局文件中加入几个属性就可以了：
{% highlight xml %}
<TextView
	android:ellipsize="marquee"
	android:focusable="true"
	android:focusableInTouchMode="true"
	android:marqueeRepeatLimit="marquee_forever"
	android:singleLine="true" />
{% endhighlight %}
2.多个跑马灯效果
</br>
在同一个layout中，两个TextView如果都设置了焦点，只有后一个会处于focused的状态，这个时候需要写一个类来继承TextView，稍作修改即可：
{% highlight java %}
public class MarqueeView extends TextView {
	public MarqueeView(Context context) {
		this(context, null);
	}

	public MarqueeView(Context context, AttributeSet attrs) {
		this(context, attrs, 0);
	}

	public MarqueeView(Context context, AttributeSet attrs, int defStyleAttr) {
		super(context, attrs, defStyleAttr);
	}

	@Override
	protected void onFocusChanged(boolean focused, int direction, Rect previouslyFocusedRect) {
		super.onFocusChanged(true, direction, previouslyFocusedRect);
	}

	@Override
	public void onWindowFocusChanged(boolean hasWindowFocus) {
		super.onWindowFocusChanged(true);
	}

	@Override
	public boolean isFocused() {
		return true;
	}
}
{% endhighlight %}
然后就可以在布局文件中使用了：
{% highlight xml %}
<MarqueeView
	android:ellipsize="marquee"
	android:marqueeRepeatLimit="marquee_forever"
	android:singleLine="true" />
{% endhighlight %}
3.在appwidget中实现跑马灯效果
</br>
appwidget比较特殊，它只支持几个固定的view，所以就不能实现多个跑马灯效果，只能实现单个效果，不过和在普通layout中的实现稍微有所不同：
{% highlight xml %}
<TextView
	android:ellipsize="marquee"
	android:focusable="true"
	android:focusableInTouchMode="true"
	android:marqueeRepeatLimit="marquee_forever"
	android:singleLine="true">
	<requestFocus
		android:duplicateParentState="true"
		android:focusable="true"
		android:focusableInTouchMode="true"/>
</TextView>
{% endhighlight %}
需要在TextView中加入requestFocus标签才可以实现跑马灯效果。