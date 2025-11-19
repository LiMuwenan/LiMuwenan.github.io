---
layout: page
title: 时间归档
permalink: /archive/
---

# 按时间归档

{% assign posts_by_year = site.posts | group_by_exp: "post", "post.date | date: '%Y'" %}

{% for year in posts_by_year %}
## {{ year.name }}年 ({{ year.items.size }}篇)
<ul class="archive-list">
  {% for post in year.items %}
  <li>
    <span class="post-date">{{ post.date | date: "%m月%d日" }}</span>
    <a href="{{ post.url }}">{{ post.title }}</a>
    {% if post.categories %}
    <span class="post-categories">
      [{% for category in post.categories %}{{ category }}{% unless forloop.last %}, {% endunless %}{% endfor %}]
    </span>
    {% endif %}
  </li>
  {% endfor %}
</ul>
{% endfor %}

<style>
.archive-list {
  list-style: none;
  padding: 0;
}

.archive-list li {
  margin: 8px 0;
  padding: 5px 0;
  border-bottom: 1px solid #eaecef;
}

.post-date {
  display: inline-block;
  width: 70px;
  color: #586069;
  font-weight: bold;
}

.post-categories {
  font-size: 0.8em;
  color: #6a737d;
  margin-left: 10px;
}
</style>