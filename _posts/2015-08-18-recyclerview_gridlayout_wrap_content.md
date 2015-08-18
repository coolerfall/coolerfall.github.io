---
layout: post
title: RecyclerView使用GridLayoutManager时不能wrap_content
head: 用RecyclerView一段时间了，最近需要实现一个自定义的数字键盘，需要放在底部，结果发现RecyclerView并不能wrap_content。
category: android
tags: [widget]
---
{% include cooler/setup %}

RecyclerView与ListView有所不同，它并不负责高度的控制，真正控制高度的是LayoutManger，但是不管是LinearLayoutManager还是GridLayoutManager都是默认将高度设置为了match_parent，所以RecyclerView中去设置wrap_content并不能实现。要想控制高度，只有写一个类继承LinearLayoutManager或者GridLayoutManager，在onMeasure中去重新设置高度。然而通过各种百度google后发现，只有[android-linear-layout-manager][1]可以勉强解决LinearLayoutManager下的wrap_content问题，GridLayoutManager没能找到合适的解决方法，最后使用了一种比较暴力的方法，直接给高度设置一个固定高度，比如设置为屏幕宽度：

{% highlight java %}
public class KeyGridLayoutManager extends GridLayoutManager {
		public KeyGridLayoutManager(Context context, int spanCount) {
			super(context, spanCount);
		}

		@Override
		public void onMeasure(RecyclerView.Recycler recycler, RecyclerView.State state, int widthSpec, int heightSpec) {
			int measuredWidth = View.MeasureSpec.getSize(widthSpec);
			int measuredHeight = View.MeasureSpec.getSize(widthSpec);
			setMeasuredDimension(measuredWidth, measuredHeight);
		}
	}
}
{% endhighlight %}
RecyclerView很灵活，但某些方面现在还不能满足开发需求，所以在开发时按需求来进行取舍。

[1]: https://github.com/serso/android-linear-layout-manager