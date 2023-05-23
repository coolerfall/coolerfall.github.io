---
title: 在Linux下进行开发工作（二）
date: 2018-10-18 17:37:14
categories: misc
tags: linux
---

{% post_link work-on-linux-part-one %}中记录了Manjaro的安装过程以及源设置等等，这一篇主要记录下常用软件的安装等等。Manjaro固然非常的方便，各种开发工具一条命令就可以安装好，但有些Windows常用软件Linux上也没有，有的可以使用Linux上的替代品，不过还是有少许软件不可避免的要使用（如微信，QQ等），这会给我们的工作带来了不少的麻烦，刚从Windows转Linux会有诸多的不习惯，不过都还好，我们总是有各种各样的办法来折腾Linux来适合我们用来做日常的开发桌面。还是那句话，如果没准备好用Linux作为日常开发桌面，千万不要尝试 : )。
<!-- more -->


#### 输入法
毕竟日常还是使用中文，中文输入法是必须得安装的，Linux下的输入框架常用的有`fcitx`和`ibus`，我选用的是[fcitx][1]：
* 安装fcitx
    ```shell
    $ sudo pacman -S fcitx fcitx-configtool 
    ```
* 安装输入法引擎
    fcitx默认自带了中文输入引擎，其他选择就比较多了，（比如国内常用的sougou拼音），我选择的是[Rime][2]，配上一个`fcitx-skin-material`用来非常不错。不过Rime默认是繁体输入，如果要改为默认简体，需新增一个自定义配置文件`$HOME/.config/fcitx/rime/
    luna_pinyin.custom.yaml`：
    ```yml
    patch:
    "menu/page_size": 9
    switches:                   # 注意缩进
        - name: ascii_mode
        reset: 0                # reset 0 的作用是当从其他输入法切换到本输入法重设为指定状态
        states: [ 中文, 西文 ]   # 选择输入方案后通常需要立即输入中文，故重设 ascii_mode = 0
        - name: full_shape
        states: [ 半角, 全角 ]   # 而全／半角则可沿用之前方案的用法。
        - name: simplification
        reset: 1                # 增加这一行：默认启用「繁→簡」转换。
        states: [ 漢字, 汉字 ]
    ```
    这里面还包括候选字数，全/半角等配置。
* 安装输入法模组
    要在桌面上使用（包括浏览器，应用等等），还需要安装输入法模组，尽可能的安装完全：
    ```shell
    $ sudo pacman -S fcitx-gtk2 fcitx-gtk3 fcitx-qt4 fcitx-qt5
    ```
* 添加配置
    在`$HOME/.xprofile`中添加如下配置
    ```text
    export GTK_IM_MODULE=fcitx
    export QT_IM_MODULE=fcitx
    export XMODIFIERS=@im=fcitx
    ```
最后重启电脑，既可以使用输入法了，关于快捷键和字体大小，可在fcitx config GUI里面配置。至此，输入法基本上就搞定了。

#### 微信
日常中微信使用的频率比较高，不可避免的要安装，一般有几种解决方案：
* [electronic-wechat][3]
使用的是微信web版，然后用electronic封装的，作者似乎以及停更了，安装后体验了下，弃了。
* [weweChat][4]
界面做得比较好看，和electronic-wechat使用同样的原理，没体验过。
* [deepin-wechat][5]
使用deepin自己定制的[wine][6]，安装后，不知道为什么我电脑上问题挺多（比如不能贴图），遂弃之。

体验过这么多之后，最后还是决定使用wine版本的，不过是自己进行配置。在github上找到一个专门针对国内软件的wine脚本[winetricks-zh][7]，会自动安装一些依赖，最后尝试下来这个版本体验最好，除了不能截图（当然有替代方案，[shutter][8]），其他基本和Windows上没太大差别。
* 安装wine及相关依赖
    ```shell
    $ sudo pacman -S wine wine-mono wine_gecko
    ```
* 使用winetricks-zh安装微信
    作者有较长时间未更新了，目前微信官网的微信已更新，会导致文件校验失败，不过只需要把脚本中的hash值改一下即可（文件为在`winetricks-zh/verb/wechat.verb`），下载官网最新安装包，然后计算sha256:
    ```shell
    $ sha256sum WeChatSetup.exe
    ```
    最后将hash值替换为此hash值即可，后续就会自动安装一些依赖。
* 配置
    安装完后可以直接打开使用，不过由于屏幕适配以及字体问题，可能会导致乱码，字体小的问题，因此我们还需要修改些配置才能正常使用。
    - 字体安
    需要将`微软雅黑`和`新宋`字体复制到`$HOME/.local/share/fonts`下，并刷新字体缓存：
    ```shell
    $ fc-cache -fv
    ```
    - 修改`$HOME/.wine/system.reg`
    找到`LogPixels`，将值修改为80（根据自己的屏幕调整）
    - 修改`$HOME/.wine/driver_c/windows/win.ini`
    在文件尾新增：
    ```text
    [Desktop]
    menufontsize=13
    messagefontsize=13
    statusfontsize=13
    IconTitleSize=13
    ```
    - 新增`$HOME/.wine/zh.reg`
    在文件中添加以下参数：
    ```text
    REGEDIT4

    [HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\FontSubstitutes]

    "Arial"="simsun"
    "Arial CE,238"="simsun"
    "Arial CYR,204"="simsun"
    "Arial Greek,161"="simsun"
    "Arial TUR,162"="simsun"
    "Courier New"="simsun"
    "Courier New CE,238"="simsun"
    "Courier New CYR,204"="simsun"
    "Courier New Greek,161"="simsun"
    "Courier New TUR,162"="simsun"
    "FixedSys"="simsun"
    "Helv"="simsun"
    "Helvetica"="simsun"
    "MS Sans Serif"="simsun"
    "MS Shell Dlg"="simsun"
    "MS Shell Dlg 2"="simsun"
    "System"="simsun"
    "Tahoma"="simsun"
    "Times"="simsun"
    "Times New Roman CE,238"="simsun"
    "Times New Roman CYR,204"="simsun"
    "Times New Roman Greek,161"="simsun"
    "Times New Roman TUR,162"="simsun"
    "Tms Rmn"="simsun"
    ```
重新打开微信，应该就没什么问题了。关于快捷键问题，Linux下无法直接使用微信中的快捷键（其实是没有焦点，如果打开微信的设置页面，再使用打开快捷键，发现还是有用的，当然这非常不方便），最后找了个办法，在系统快捷键中，添加一个快捷键来打开微信即可，命令使用wine来打开：
```text
env WINEPREFIX="/home/cooler/.wine" wine "/home/cooler/.wine/drive_c/Program Files/Tencent/WeChat/WeChat.exe"
```
这样就可以使用快捷键打开微信，不过不能关闭，只能使用`Esc`来关闭，不过这点小问题不影响使用。至于其他软件，可以参考`winetricks-zh`。

#### 开发软件
一般开发软件都有Linux版本的，除非是系统特有的软件（如Windows的visual studio等等），这种就没办法了，只能放弃使用Linux了。

#### 截图软件
Linux下首推[shutter][8]，功能已经非常完善了，在系统快捷键中新增一个，命令如下：
```text
shutter -s
```
然后配合微信，QQ等使用，直接粘贴到对话框中即可。

#### 办公软件
Windows下办公软件莫过于Office全家桶，但Linux下就没这么好的办公软件了，替代品为[WPS Office][9]和[Libre Office][10]，wps用起来比较接近MS Office，这个看个人喜好选择。

#### 其他
* 制图软件[GIMP][11]
* 思维导图[XMind][12]
* PDF阅读使用自带的`Atril Document Viewer`就可以了
* 视频播放软件[VLC][13]。
* 邮件客户端[Evolution][14]
* ......

[1]: https://wiki.archlinux.org/index.php/Fcitx
[2]: https://wiki.archlinux.org/index.php/Rime_IME
[3]: https://github.com/geeeeeeeeek/electronic-wechat
[4]: https://github.com/trazyn/weweChat
[5]: https://aur.archlinux.org/packages/deepin-wechat/
[6]: https://www.winehq.org/
[7]: https://github.com/hillwoodroc/winetricks-zh
[8]: http://shutter-project.org/
[9]: https://www.wps.com/
[10]: https://www.libreoffice.org/
[11]: https://www.gimp.org/
[12]: https://www.xmind.net/
[13]: https://www.videolan.org/vlc/
[14]: https://wiki.gnome.org/Apps/Evolution