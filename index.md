---
layout: default
title: 首页
---

欢迎来到我的博客！这是基于 GitHub Pages + Jekyll 的最小博客示例。

最近文章：
<ul>
{% for post in site.posts %}
  <li><a href="{{ post.url }}">{{ post.title }}</a> （{{ post.date | date: "%Y-%m-%d" }}）</li>
{% endfor %}
</ul>

要写新文章：在仓库中新建文件夹 `_posts`，文件名格式 `YYYY-MM-DD-标题.md`，下面有示例。
