---
layout: default
title: 所有文章
permalink: /posts/
---

<div class="post-list">
  <h2>文章归档</h2>
  <p>这里记录了我的所有技术思考和心得：</p>
  
  <ul>
    {% for post in site.posts %}
      <li style="margin-bottom: 15px; border-bottom: 1px solid #eee; padding-bottom: 10px;">
        <span style="color: #888; font-size: 0.9em;">{{ post.date | date: "%Y-%m-%d" }}</span>
        <br>
        <a href="{{ post.url }}" style="font-size: 1.1em; font-weight: bold; text-decoration: none; color: #2c3e50;">
          {{ post.title }}
        </a>
      </li>
    {% endfor %}
  </ul>
</div>
