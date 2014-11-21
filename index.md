---
layout: page
title: Halt in Air
tagline:
---
{% include JB/setup %}

Scratching to build my blog, [Halt in Air](http://malikey.github.io), within the help of [Jekyll-Bootstrap](http://jekyllbootstrap.com).


<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
