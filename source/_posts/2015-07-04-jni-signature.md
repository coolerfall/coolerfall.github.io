---
layout: post
title: Android JNI类型、方法签名规范
category: android
tags: [jni]
---

在Android开发中不免会使用到JNI，JNI编程中可以使用javah等工具自动生成jni的头文件，但是如果想自己手动注册，那么就需要了解JNI方法的签名规范，记录一下，以备以后查看。
<!-- more -->

| Java类型 |  Native类型  | JNI签名 |
| :------: | :----------: | :-----: |
| boolean  |   jboolean   |    Z    |
|   byte   |    jbyte     |    B    |
|   char   |    jchar     |    C    |
|  short   |    jshort    |    S    |
|   int    |     jint     |    I    |
|   long   |    jlong     |    J    |
|  float   |    jfloat    |    F    |
|  double  |   jdouble    |    D    |
|  byte[]  |  jbyteArray  |   [B    |
|  char[]  |  jcharArray  |   [C    |
| short[]  | jshortArray  |   [S    |
|  int[]   |  jintArray   |   [I    |
|  long[]  |  jlongArray  |   [L    |
| float[]  | jfloatArray  |   [F    |
| double[] | jdoubleArray |   [D    |
| Java类(例: String) | jstring/jobject | L全类名;(例: Ljava/lang/String; |
| Java方法(例: start(String path, long pos, long duration)) | Native方法(例: start(jstring path, jlong pos, jlong duration)) | (参数签名...)返回值签名(例: (Ljava/lang/String;JJ)V) |

要注意的是java类的签名，最后的分号不要忘记。String类比较特别，jni提供了就jstring与之对应，java方法签名中，括号里面是所有参数的类型签名，中间无间隔，括号外面是返回值类型签名。