---
title: 在Linux下进行开发工作（三）
date: 2018-10-22 10:52:13
categories: misc
tags: linux
---

{% post_link work-on-linux-part-two %}中记录了Manjaro日常使用中的一些常用软件和配置。基本可以在Linux进行开发了，不过由于屏幕适配以及字体的问题，会出现字体发虚，太小等等问题（4k屏以上一般来说不会有这些问题，但也不排除有个别情况），在Manjaro上的中文展示不是特别理想。这一篇主要记录下如何通过配置来调优字体显示，以适配自己的电脑。依然那句话，如果没准备好用Linux作为日常开发桌面，千万不要尝试 : )。
<!-- more -->


#### 字体安装
首先找一个能够对中文展示比较友好字体，几番寻找，找到了`Noto Sans CJK`，也有人使用`文泉驿微米黑`，不过我试用了几天，还是觉得`Noto`系列看起来舒服些（这就看个人喜好了），这些字体都可以直接使用`pacman`安装：
```shell
$ sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji wqy-microhei  
```

#### 配置字体
在`$HOME/.config/fontconfig/fonts.conf`（若没有则创建，若存在则覆盖）中添加如下配置：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">

<fontconfig>

  <match target="font">
    <edit mode="assign" name="rgba">
      <const>rgb</const>
    </edit>
  </match>

  <match target="font">
    <edit mode="assign" name="hintstyle">
      <const>hintfull</const>
    </edit>
  </match>

  <match target="font">
    <edit mode="assign" name="antialias">
      <bool>true</bool>
    </edit>
  </match>

  <match target="font">
    <edit name="lcdfilter" mode="assign">
      <const>lcddefault</const>
    </edit>
  </match>

  <!-- Default font (no fc-match pattern) -->
  <match>
    <edit mode="prepend" name="family">
      <string>Noto Sans Mono</string>
    </edit>
  </match>

  <!-- Default font for the zh_CN locale (no fc-match pattern) -->
  <match>
    <test compare="contains" name="lang">
      <string>zh_CN</string>
    </test>
    <edit mode="prepend" name="family">
      <string>Noto Sans CJK SC</string>
    </edit>
  </match>

  <!-- Default sans-serif font -->
  <match target="pattern">
    <test qual="any" name="family">
      <string>sans-serif</string></test>
    <edit name="family" mode="prepend" binding="same">
      <string>Noto Sans</string>
    </edit>
  </match>

  <!-- Default serif fonts -->
  <match target="pattern">
    <test qual="any" name="family">
      <string>serif</string>
    </test>
    <edit name="family" mode="prepend" binding="same">
      <string>Noto Serif</string>
    </edit>
  </match>

  <!-- Fallback fonts preference order -->
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>Noto Sans</family>
      <family>Noto Sans CJK SC</family>
      <family>Noto Sans CJK TC</family>
      <family>Noto Sans CJK JP</family>
      <family>Noto Sans CJK KR</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>serif</family>
    <prefer>
      <family>Noto Serif</family>
      <family>Noto Serif CJK SC</family>
      <family>Noto Serif CJK TC</family>
      <family>Noto Serif CJK JP</family>
      <family>Noto Serif CJK KR</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>monospace</family>
    <prefer>
      <family>Noto Sans Mono</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
    </prefer>
  </alias>

</fontconfig>
```
然后刷新字体缓存：
```shell
$ fc-cache -fv
```

#### 桌面字体设置
* 在`菜单->首选项->外观->字体`中可以找到桌面相关字体配置，这里将这些字体（除等宽）都改为`Noto Sans CJK SC Regular`，等宽字体改为`Noto Sans Mono Regular`，字体大小设置为10左右，当然这些根据个人的电脑的效果调整。
* 在渲染中的细节中，关闭分辨率的自动检测，然后手动设置DPI，我的电脑是1080的，故设置为124左右即可（其他屏幕可视效果调整）；微调设置为完全，其他的可以保持不动。

#### Qt应用字体设置
* 在`菜单->首选项->Qt5设置`中，将字体设置为`Noto Sans CJK`系列，根据效果调整大小。

#### 浏览器字体设置
浏览器页面的字体不受桌面字体设置的影响，桌面字体仅仅只能改变浏览器顶部选项卡以及菜单的字体大小，网页的字体还需要在浏览器中单独设置，这里以chrome为例，其他浏览器有类似设置。打开chrome设置页面，找到自定义字体，然后打开高级字体设置。浏览器的配置经过反复的尝试折腾，最终配置如下:
![img][1]
以上为`Default`的配置，还需要配置下`Simplified Han`，在Script中找到并修改成一样即可。如果有emoji显示不正常的问题，需检查[字体配置](#配置字体)是否正确，最后看下我的配置效果：
![img][2]
![img][3]
至于中文字体的效果，可根据自己的喜好选择Noto系列或者文泉驿字体，或者其他好看的中文字体。

#### 其他
至于其他应用（如：IDEA，VS Code等等），内部都有相应设置字体的地方，按自己的喜好设置字体及大小即可。


[1]: /images/work-on-linux/font-config.png
[2]: /images/work-on-linux/font-config-effects1.png
[3]: /images/work-on-linux/font-config-effects2.png