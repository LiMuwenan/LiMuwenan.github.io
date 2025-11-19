---
layout: page
title: 文章目录
permalink: /categories/
---

# 文章目录

这里是我所有的知识整理，按分类组织。

{% for category in site.categories %}
## {{ category[0] }} ({{ category[1].size }}篇)
<div class="category-posts">
  {% for post in category[1] %}
  <div class="post-item">
    <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
    <div class="post-meta">
      <span class="date">{{ post.date | date: "%Y年%m月%d日" }}</span>
      {% if post.description %}
      <span class="description">- {{ post.description }}</span>
      {% endif %}
    </div>
  </div>
  {% endfor %}
</div>
{% endfor %}

<style>
.category-posts {
  margin: 20px 0;
  border-left: 3px solid #0366d6;
  padding-left: 15px;
}

.post-item {
  margin: 15px 0;
  padding: 10px;
  background: #f6f8fa;
  border-radius: 5px;
}

.post-item h3 {
  margin: 0 0 5px 0;
}

.post-item h3 a {
  color: #0366d6;
  text-decoration: none;
}

.post-item h3 a:hover {
  text-decoration: underline;
}

.post-meta {
  font-size: 0.9em;
  color: #586069;
}

.date {
  font-weight: bold;
}

.description {
  font-style: italic;
}
</style>