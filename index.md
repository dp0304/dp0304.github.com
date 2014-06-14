---
layout: page
title: 救药无方 
tagline: Hey you !!! You may say that I'm a dreamer, but I' m not the only one !
---
{% include JB/setup %}



<ul class="posts">
{% for post in site.posts %}
<li><p class="date" cate="{{ post.categories }}">{{ post.date | date:"%Y-%m-%d" }}</p> <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>

