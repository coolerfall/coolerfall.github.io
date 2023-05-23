---
title: 迁移评论系统disqus到colina
date: 2019-02-19 16:45:50
category: misc
tags: [colina, comment]
---

自从迁移至Hexo后，就开始折腾评论系统，现目前有比较多的评论系统，如`disqus`、`来必力`、`gitment`等等， 经过一番比较，选择了老牌的评论系统`disqus`，用起来还是非常不错的。但是用了一段时间，由于disqus需要科学上网才能使用，对于国内的用户不太友好，于是乎开始寻找其他的解决方案，找到了一些可以自己搭建的博客系统，发现多多少少不能满足需求，最终还是决定自己来造个轮子，方便自己管理评论系统，又可以保证数据不丢失。于是利用空闲时间，花了些时间折腾出了[colina][1]，一款轻量、简单易用、支持Markdown的评论系统。

<!-- more -->

#### 部署Colina
现目前colina的主要功能已开发完成，但还有不少细节未完善，暂时只提供了`docker`的部署方式，后续将逐步完善。
```shell
$ docker pull coolerfall/colina:latest
```
使用`docker compose`来编排，方便启动/关闭：
```yml
version: 3

services:
  colina:
    restart: always
    image: coolerfall/colina:latest
    depends_on:
      - db
    environment:
      - COLINA_URL=http://localhost:3000
      - COLINA_DSN=postgres://colina:colinapass@db/colina?sslmode=disable
      - COLINA_DATA_DIR=/data
    ports:
      - 3000:3000
    volumes:
      - /home/colina:/data
  db:
    restart: always
    image: postgres:11-alpine
    environment:
      - POSTGRES_USER=colina
      - POSTGRES_PASSWORD=colinapass
      - POSTGRES_INITDB_ARGS="--encoding=UTF8"
    volumes:
      - /home/colina/db:/var/lib/postgresql/data
```
然后使用`docker-compose`命令启动起来即可：
```shell
$ docker-compose -f docker-compose.yml up -d 
```
在浏览器中打开`http://localhost:3000`，显示管理登录页面即表示运行正常，根据提示配置后台相应参数。

#### 在hexo中接入Colina（以[next][2]主题为例）
* 在管理页面新增一个站点，获取到 `site id`。 -->

* 在`themes/next/layout/_third-party`中新增`colina.swig`文件，并添加以下内容：
```swig
  {% if theme.colina.enable %}
  <link rel="stylesheet" href="{{ theme.colina.url }}/embed/colina.min.css">

  {% if theme.disqus.count %}
    <script type="text/javascript">
      var colinaConfig = {
        siteId: "{{ theme.colina.site_id }}",
        apiUrl: "{{ theme.colina.url }}",
        lang: "{{ theme.colina.language }}" || navigator.language || 
          navigator.systemLanguage || navigator.userLanguage
      };
      (function() {
          var d = document, s = d.createElement("script");
          s.src = "{{ theme.colina.url }}/embed/counter.min.js";
          (d.head || d.body).appendChild(s);
        })();
    </script>
  {% endif %}

  {% if page.comments %}
    <script type="text/javascript">
      var colinaConfig = {
        identifier: "{{ page.path }}",
        siteId: "{{ theme.colina.site_id }}",
        apiUrl: "{{ theme.colina.url }}",
        title: "{{ page.title| addslashes }}",
        lang: "{{ theme.colina.language }}" || navigator.language || 
          navigator.systemLanguage || navigator.userLanguage
      };
      function loadComments() {
        (function() {
          var d = document, s = d.createElement("script");
          s.src = "{{ theme.colina.url }}/embed/colina.min.js";
          (d.head || d.body).appendChild(s);
        })();
      }

      {% if theme.colina.lazyload %}
        $(function () {
          var offsetTop = $('#comments').offset().top - $(window).height();
          if (offsetTop <= 0) {
            // load directly when there's no a scrollbar
            loadComments();
          } else {
            $(window).on('scroll.colina_scroll', function () {
              // offsetTop may changes because of manually resizing browser window or lazy loading images.
              var offsetTop = $('#comments').offset().top - $(window).height();
              var scrollTop = $(window).scrollTop();

              // pre-load comments a bit? (margin or anything else)
              if (offsetTop - scrollTop < 60) {
                $(window).off('.colina_scroll');
                loadComments();
              }
            });
          }
        });
      {% else %}
        loadComments();
      {% endif %}
    </script>
  {% endif %}

{% endif %}
```

* 在同目录下的`index.swig`中追加：
```swig
{% include 'colina.swig' %}
```

* 在`themes/next/layout/_partials/comments.swig`中追加以下内容，用于渲染评论内容：
```swig
{% elseif theme.colina.enable %}
    <div class="comments" id="comments">
      <div id="colina-comments">
        <noscript>
          Please enable JavaScript to view comments powered by Colina.
        </noscript>
      </div>
    </div>
```

* 在`themes/next/layout/_macro/post.swig`中添加以下内容，用于展示评论条数：
```swig
  {% if post.comments %}
  ...
  ...

    {% elseif theme.colina.enable and theme.colina.count %}
      <span class="post-comments-count">
        <span class="post-meta-divider">|</span>
        <span class="post-meta-item-icon">
          <i class="fa fa-comment-o"></i>
        </span>
        <a href="{{ url_for(post.path) }}#comments" itemprop="discussionUrl">
          <span class="post-meta-item-text">{{ __('post.comments_count') + __('symbol.colon') }}</span>
          <span class="post-comments-count colina-comment-count"
                  data-colina-identifier="{{ post.path }}" itemprop="commentCount"></span>
        </a>
      </span>
```

* 在`themes/next/_config.yml`中添加配置项：
```yml
colina:
  enable: true
  site_id: your site id
  url: http://localhost:3000
  count: true
  lazyload: false
  language:
```

* 最后运行hexo即可看到显示效果

[1]: https://hub.docker.com/r/coolerfall/colina
[2]: https://github.com/theme-next/hexo-theme-next
