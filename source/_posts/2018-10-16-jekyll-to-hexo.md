---
title: 从Jekyll迁移到Hexo
category: misc
tags: [jekyll, hexo]
date: 2018-10-16 15:41:22
---

自上次更新博客以来已超过2年，其一是工作太忙，其二主要是因为自己太懒，不过这其间学习非常多的新知识。最近新增了几个项目之后，发现已经太久没更新博客，同时感觉jekyll用起来还是不够舒服，经过一番google，找到[hexo][1]，hexo原理和jekyll差不多，但是代码块高亮直接使用的markdown的语法，无需像jekyll那样的高亮语法，又找了个简洁的主题[hexo-theme-next][2],于是折腾起hexo来，至于为什么不去折腾hugo，主要是因为找了半天主题没找到满意的，遂放弃了。hexo的使用可以参见[官网][3]的步骤，非常的详细，大致记录一下迁移过程。
<!-- more -->
#### 项目初始化
```shell
$ npm install -g hexo-cli
$ hexo init coolerfall.github.io
$ cd coolerfall.github.io && npm install
```

#### 配置
完全配置文件可参考官网的文档，一般来说需要改下作者信息，描述等等，默认是使用landscape主题，我这里使用的是[hexo-theme-next][2]，参见文档将代码加入hexo目录下的`themes`下，修改根目录`_config.yml`中的`theme`为`next`即可，next主题完全配置参见官网文档，需要注意的是next默认没有生成`tags`和`categories`的index页面，需要自己手动生成：
```shell
$ hexo new page tags
$ hexo new page categories
```
生成的文件在`_posts`下的`tags`和`categories`中，将`type`分别改为`tags`和`categories`即可，后面自动生成。配置好后，可以简单运行下看看效果：
```shell
$ hexo serve
```

#### 迁移
由于之前用的是Jekyll，post文章的格式为`:year-:month-:day-:title.md`，因此hexo也采用同样的格式即可，只需将`new_post_name`改为这种格式。将Jekyll目录下的`_posts`中文章全部copy到hexo目录下的`source/_posts`中，由于文章是Markdown格式，所以基本没有太大改动，我之前使用了Jekyll的一个高亮插件，写法与Markdown有所区别，只需把这些写法改为Markdown的写法即可，然后运行可看到效果。关于文章长度太长，需显示`阅读全文`，有两种方式：
* 自动截取
需修改`next`主题配置文件中`auto_excerpt`为true，以及截取开始长度（但这种方式不推荐）。
* 手动添加
只需要在每篇post中需要截取的地方加入`<!-- more -->`即可。

#### 部署
最后部署至github，hexo有多种部署方式，可部署至`Git`，`Heroku`，`Netlify`等等，都有相应部署工具。这里使用`hexo-deployer-git`，需在根目录下加入
```shell
$ npm install hexo-deployer-git --save
```
在根目录配置文件中的`deploy`中加入相关配置，然后执行部署
```shell
$ hexo g -d
```
注意，Hexo与Jekyll有区别，github默认在服务器上生成并部署，而Hexo需要我们在本地生成好静态文件后再上传，因此，需要分两个分支来存放代码，master分支放静态文件，另起一个分支（如：source）来存放源码，这里就需要在deploy中加入这两个分支的操作：
```yml
deploy:
  - type: git
    repo: git@github.com:you/your.github.io.git
    branch: [master]
  - type: git
    repo: git@github.com:you/anothergit.git
    branch: [master]
    extend_dirs: /
    ignore_hidden: false
    ignore_pattern:
        public: .
```
至此，博客从Jekll成功迁移到了Hexo。

[1]: https://github.com/hexojs/hexo
[2]: https://github.com/theme-next/hexo-theme-next
[3]: https://hexo.io/zh-cn/docs/