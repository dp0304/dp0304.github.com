---
layout: page
title: 救药无方 
tagline: 当大浪潮压过来时候，我会竖中指，我会还手。
---
{% include JB/setup %}



<ul class="posts">
{% for post in site.posts %}
<li><p class="date" cate="{{ post.categories }}">{{ post.date | date:"%Y-%m-%d" }}</p> <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>

