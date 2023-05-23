---
title: 在Linux下进行开发工作（一）
date: 2018-10-17 17:47:03
categories: misc
tags: linux
---

之前在做嵌入式开发时，经常会使用到Linux，但都是使用的虚拟机来安装Linux，然后在Windows上通过ssh连接，使用的命令行方式，并未使用Linux当桌面。后来做Android开发后，基本是在Windows下进行开发工作的，部分Linux下的命令工具可以通过[mingw][1]来使用，但终究还是不如直接在Linux下来得方便。年中的时候，入手了一台新本，开始折腾起来Linux，选择的是[Manjaro][2]发行版的Mate Desktop，几个桌面版本尝试后，最终选择了Mate。Manjaro是[Arch Linux][3]衍生版，但是简化了安装过程，几乎是一键安装，省去了Arch Linux的繁琐配置，但却拥有Arch Linux同样丰富的软件库。折腾过程中遇到不少问题，都通过各种google解决了，如果没准备好用Linux作为日常开发桌面，千万不要尝试 : )。
<!-- more -->


#### 安装Manjaro
在[Manjaro官网][2]找自己比较喜欢的桌面下载镜像文件，准备一个空的U盘（后面会被格式化），Windows下使用[rufus][4]来创建USB启动盘。将镜像文件烧写到U盘（注意：rufus中需选择DD模式），电脑上选择U盘启动，就会进入到Manjaro的安装界面，选择对应的语言，驱动等等，然后启动可以进入到Manjaro系统进行体验，直接进入安装步骤，都是些个性化设置。在分区的时候需要注意，选择手动分区，如果想要安装双系统，需要找到Windows的`efi`分区（一般是一个100M的FAT32分区），然后挂载`/boot/efi`（不要选择格式化），其他分区按需分配即可。我的分区结构：
* `/`分区（必须有此分区） --- 50G
* `/boot`分区 --- 512M
* `/var`分区（pacman会缓存安装包） --- 30G
* `/swap`分区 --- 4G
* `/home`分区 --- 余下的所有空间
最后到安装等待界面，待安装完成重新启动应该会有个GRUB bootloader了。但我的电脑死活没看到grub，最后看到[官网安装教程][5]才知道，有些电脑装双系统后，GRUB会显示不出来，解决方案就是安装[rEFInd][6]，这步操作需要重启电脑，通过USB启动，这里会显示出刚刚安装的系统（不要选择安装在U盘的那个系统，其实就是通过U盘来做引导），选择进入，然后在Terminal中安装rEFInd：
```shell
$ sudo pacman -S refind-efi
```
安装完后重启，这次就看到rEFInd界面了，在这里可以选择启动Windows或者Manjaro。

#### 更新系统
首次进入系统，第一件事情就是更新系统，首先我们需要做的是设置国内的镜像源，这样下载的速度会快些。
* 配置镜像源
    ```
    $ sudo pacman-mirrors -i -c China -m rank
    ```
    这条命令会获取中国的镜像源并进行排序，最后生成一个镜像列表，然后选择需要的源地址。
* 设置archlinux源
    打开`/etc/pacman.conf`，并添加一下内容
    ```text
    [archlinuxcn]
    SigLevel = Optional TrustedOnly
    Server = http://repo.archlinuxcn.org/$arch
    ```
    然后根据上面的配置，生成一个新的`mirrolist`：
    ```shell
    $ sudo pacman-mirrors -g
    ```
* 更新
    镜像里面设置完成后，就进行系统的全面升级:
    ```shell
    $ sudo pacman -Syyu
    ```
    由于使用了archlinuxcn的镜像，还需要安装`archlinuxcn-keyring`，才能安装镜像上的软件：
    ```shell
    $ sudo pacman -S archlinuxcn-keyring
    ```
平时使用中还会安装一些非官方软件（也就是AUR包），这需要安装一个额外的工具，一般安装的`yaourt`，当然还有其他很多[AUR工具][7]，可以根据喜好自行选择。如果不习惯使用命令行安装，也可以使用pacman的GUI，在首选项中可以设置打开AUR，然后搜索时候选择AUR即可安装相应的AUR包。


[1]: http://www.mingw.org/
[2]: https://manjaro.org/get-manjaro/
[3]: https://www.archlinux.org/
[4]: http://rufus.ie/
[5]: https://wiki.manjaro.org/index.php?title=UEFI_-_Install_Guide
[6]: http://www.rodsbooks.com/refind/index.html
[7]: https://wiki.archlinux.org/index.php/AUR_helpers