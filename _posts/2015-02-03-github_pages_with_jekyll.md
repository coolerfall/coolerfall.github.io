---
layout: post
title: Windows下使用Jekyll在github pages搭建博客
head: 一直想搭建一个博客来记录些开发中遇到的问题，但是自己对前端不熟悉以及服务器主机等等原因，迟迟没有行动起来。后来发现了github pages这个东西，可以支持用户在github上搭建个人博客，于是乎开始折腾起Jekyll了。
category: web
tags: [github pages, jekyll, ruby]
---
{% include cooler/setup %}

### Jekyll的简单教程 ###
1.Jekyll是基于Ruby开发的，所以先安装[rubyinstaller以及DevKit][1]，要注意DevKit需下载与rubyinstaller对应的版本。安装好ruby之后，将DevKit解压，在cmd中切换到DevKit的根目录，执行`$ ruby dk.rb init`生成config.yml配置文件，然后在config.yml添加ruby的安装目录：
{% highlight text %}
---
- E:/Ruby200
{% endhighlight %}

注意是*---*下面加*-*和空格最后在cmd中执行`$ ruby dk.rb install`，出现提示：
{% highlight text %}
$ ruby dk.rb install
[INFO] Updating convenience notice gem override for 'E:/Ruby200'
[INFO] Installing 'E:/Ruby200/lib/ruby/site_ruby/devkit.rb'
{% endhighlight %}
则ruby环境搭建完成了。

2.在cmd中通过gem来安装jekyll，首先使用`$ gem list`查看是否安装liquid，若未安装则执行`$ gem install liquid`，安装完成后再执行`$ gem install jekyll`来安装jekyll。如果安装速度很慢，可以考虑将ruby源更换为淘宝源：
{% highlight text %}
$ gem sources --remove https://rubygems.org/
$ gem sources -a https://ruby.taobao.org/
$ gem sources -l
*** CURRENT SOURCES ***

https://ruby.taobao.org
{% endhighlight %}

3.安装好jekyll后，从github上下载[jekyll bootstrap][2]模板:
{% highlight text %}
$ git clone https://github.com/plusjade/jekyll-bootstrap.git jekyll
$ cd jekyll
$ jekyll server
{% endhighlight %}
在浏览器中输入localhost:4000，成功的话会看到一个demo网页。

[1]: http://rubyinstaller.org/downloads/
[2]: https://github.com/plusjade/jekyll-bootstrap/
