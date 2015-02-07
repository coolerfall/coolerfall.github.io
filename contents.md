---
layout: page
title: 目录
description: ""
---
{% include cooler/setup %}

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date: " %Y.%m.%d" }}</span>  <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>