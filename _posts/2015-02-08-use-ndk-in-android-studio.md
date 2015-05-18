---
layout: post
title: 在Android Studio中利用gradle来自动编译jni
category: tools
tags: [android studio, ndk]
head: 在最近的Android开发中，项目逐渐从Eclipse迁移到Android Studio中来，google官方现在并未在Android Studio中支持ndk的开发，但是我们可以利用gradle自动编译jni。
---
{% include cooler/setup %}

### 步骤 ###

1.在新建项目中找到local.properties，在里面加入ndk的路径(ndk必须是r9以上)：
{% highlight text %}
ndk.dir=E\:\\Android\\ndk-r10d
{% endhighlight %}

2.在app\src\main中新建jni文件夹，在这里面存放要编译的c/c++文件以及Android.mk

3.在app中的build.gradle中加入两个task：ndkBuild和copyJniLibs，第一个task为ndk执行编译，第二个task将编译好的so库copy至jniLibs目录，这样才Android Studio最后打包的时候才会将so库打包进去：
{% highlight groovy %}
android {
    compileSdkVersion 21
    buildToolsVersion "21.1.2"
    defaultConfig {
        applicationId "com.xxx.yyy"
        versionCode 1
        versionName '1.0'
	    minSdkVersion 10
	    targetSdkVersion 21
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles ('proguard-android.txt')
        }
    }

    tasks.withType(JavaCompile) {
        compileTask -> compileTask.dependsOn 'ndkBuild', 'copyJniLibs'
    }

    sourceSets {
        main {
	        jni.srcDirs = []
            jniLibs.srcDirs = ['src/main/jniLibs']
        }
    }
}

task ndkBuild(type: Exec) {
	def ndkDir = project.plugins.findPlugin('com.android.application').sdkHandler.getNdkFolder()
    commandLine "$ndkDir/ndk-build.cmd", '-C', 'src/main/jni',
		    "NDK_OUT=$buildDir/ndk/obj",
		    "NDK_APP_DST_DIR=$buildDir/ndk/libs/\$(TARGET_ARCH_ABI)"
}

task copyJniLibs(type: Copy) {
    from fileTree(dir: file(buildDir.absolutePath + '/ndk/libs'), include: '**/*.so')
    into file('src/main/jniLibs')
}
{% endhighlight %}

这两个task不要放在android{}中，放在android{}外即可，否则无法编译。其中*NDK_APP_DST_DIR*为编译后的库存放的文件夹，根据需求自己设定，我在这里是设置为的buildDir下的nkd文件夹

4.最后编译，Android Studo会自动执行ndkBuild和copyJniLibs这两个task