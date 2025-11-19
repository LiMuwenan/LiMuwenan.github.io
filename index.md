---
layout: home
title: 欢迎来到我的知识库
description: 这里是我用 Markdown 记录的所有内容
---

## 最新文章

{% for post in site.posts limit:5 %}
<div class="recent-post">
  <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
  <div class="post-info">
    <span class="date">{{ post.date | date: "%Y年%m月%d日" }}</span>
    {% if post.categories %}
    <span class="categories">
      {% for category in post.categories %}
      <span class="category-tag">{{ category }}</span>
      {% endfor %}
    </span>
    {% endif %}
  </div>
  {% if post.description %}
  <p class="post-description">{{ post.description }}</p>
  {% endif %}
</div>
{% endfor %}

## 快速导航

<div class="nav-cards">
  <div class="nav-card">
    <h3><a href="/categories/">📁 分类目录</a></h3>
    <p>按分类浏览所有文章</p>
  </div>
  <div class="nav-card">
    <h3><a href="/archive/">📅 时间归档</a></h3>
    <p>按发布时间浏览文章</p>
  </div>
</div>

<style>
.recent-post {
  margin: 20px 0;
  padding: 15px;
  border: 1px solid #e1e4e8;
  border-radius: 6px;
  background: #fff;
}

.post-info {
  margin: 5px 0 10px 0;
  font-size: 0.9em;
  color: #586069;
}

.date {
  font-weight: bold;
}

.categories {
  margin-left: 10px;
}

.category-tag {
  display: inline-block;
  background: #f1f8ff;
  color: #0366d6;
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 0.8em;
  margin-right: 5px;
}

.post-description {
  color: #586069;
  font-style: italic;
  margin: 10px 0 0 0;
}

.nav-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
  margin: 30px 0;
}

.nav-card {
  padding: 20px;
  background: #f6f8fa;
  border-radius: 8px;
  text-align: center;
  border: 1px solid #e1e4e8;
}

.nav-card h3 {
  margin: 0 0 10px 0;
}

.nav-card h3 a {
  color: #0366d6;
  text-decoration: none;
}

.nav-card h3 a:hover {
  text-decoration: underline;
}

.nav-card p {
  margin: 0;
  color: #586069;
}
</style>