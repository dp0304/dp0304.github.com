---
layout: page
title: 从前慢 
tagline: 说一句 是一句 
---
{% include JB/setup %}



<ul class="posts">
{% for post in site.posts %}
<li><p class="date" cate="{{ post.categories }}">{{ post.date | date:"%Y-%m-%d" }}</p> <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>

