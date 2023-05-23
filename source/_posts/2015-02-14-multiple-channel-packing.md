---
title: Android Studio多渠道批量打包
category: tools
tags: [android studio]
---

Android市场众多，在打包App的时候需要对每个渠道添加不同的参数用于统计，但是针对每个渠道单独打包比较麻烦，所以要考虑使用批量打包。AS使用gradle来构建android项目，我们可以利用来进行批量打包操作。
<!-- more -->

### 步骤 ###

1.在manifest中找到与渠道相关的参数，增加相应的PlaceHolder，例如：
```xml
<meta-data
	android:name="aid"
	android:value="${APP_AID}" />

<meta-data
	android:name="pid"
	android:value="${APP_PID}" />
```

2.在build.gradle设置productFlavors：
```groovy
android {
	productFlavors {
		wandoujia {
			manifestPlaceholders = [APP_AID : 80, APP_PID : 43]
		}

		yingyongbao {
			manifestPlaceholders = [APP_AID : 60, APP_PID : 45]
		}
	}
}
```
manifestPlaceholders是一个数组，可以根据实际需求增减参数

3.最后在AS中Build->Generate Signed APK，最后根据提示打包APK，AS会根据productFlavors生成相应的包