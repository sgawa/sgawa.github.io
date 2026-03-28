---
layout: default
title: blog
---
# blog

{% if site.posts.size == 0 %}
*no posts yet.*
{% else %}
<ul>
{% for post in site.posts %}
  <li><a href="{{ post.url }}">{{ post.title }}</a> &mdash; <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%Y-%m-%d" }}</time></li>
{% endfor %}
</ul>
{% endif %}
