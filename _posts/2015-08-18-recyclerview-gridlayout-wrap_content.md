---
layout: post
title: RecyclerView使用GridLayoutManager时不能wrap_content
head: 用RecyclerView一段时间了，最近需要实现一个自定义的数字键盘，需要放在底部，结果发现RecyclerView并不能wrap_content。
category: android
tags: [widget]
---
{% include cooler/setup %}

## 这篇文章已废弃，support v23.2.0已支持wrap_content属性，只需要调用LayoutManager的setAutoMeasureEnabled即可
</br>
RecyclerView与ListView有所不同，它并不负责高度的控制，真正控制高度的是LayoutManger，但是不管是LinearLayoutManager还是GridLayoutManager都是默认将高度设置为了match_parent，所以RecyclerView中去设置wrap_content并不能实现。要想控制高度，只有写一个类继承LinearLayoutManager或者GridLayoutManager，在onMeasure中去重新设置高度。然而通过各种google后发现[android-linear-layout-manager][1]可以解决LinearLayoutManager下的wrap_content问题，GridLayoutManager可以采用类似的方法来实现wrap_content，需要稍微修改一下[android-linear-layout-manager][1]：

{% highlight java %}
public class KeyGridLayoutManager extends GridLayoutManager {
		....
	
	public void onMeasure(RecyclerView.Recycler recycler, RecyclerView.State state, int widthSpec, int heightSpec) {
		....

		for (int i = 0; i < adapterItemCount; i++) {
			if (vertical) {
				if (!hasChildSize) {
					if (i < stateItemCount) {
						// we should not exceed state count, otherwise we'll get IndexOutOfBoundsException. For such items
						// we will use previously calculated dimensions
						measureChild(recycler, i, widthSize, unspecified, childDimensions);
					}
				}

				/* 此处修改 */
				if (i % getSpanCount() == 0) {
					height += childDimensions[CHILD_HEIGHT];
				}

				if (i == 0) {
					width = childDimensions[CHILD_WIDTH];
				}

				if (hasHeightSize && height >= heightSize) {
					break;
				}
			} else {
				if (!hasChildSize) {
					if (i < stateItemCount) {
						// we should not exceed state count, otherwise we'll get IndexOutOfBoundsException. For such items
						// we will use previously calculated dimensions
						measureChild(recycler, i, unspecified, heightSize, childDimensions);
					}
				}

				/* 此处修改 */
				if (i % getSpanCount() == 0) {
					width += childDimensions[CHILD_WIDTH];
				}

				if (i == 0) {
					height = childDimensions[CHILD_HEIGHT];
				}

				if (hasWidthSize && width >= widthSize) {
					break;
				}
			}

		....
	}

		....
}
{% endhighlight %}
只需要修改onMeasure里面的几句代码，即可实现RecyclerView使用GridLayoutManager时wrap_content，
并且完全可以适用于ScrollView或RecyclerView嵌套RecyclerView的问题。
<br/>
<br/>
ps：这里并没有考虑SpanSizeLookup的问题，如果使用了SpanSizeLookup，此方法不再wrap_content。

[1]: https://github.com/serso/android-linear-layout-manager