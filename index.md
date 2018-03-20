---
layout: default
title: blog content
---

{% for post in site.posts %}
*    [{{ post.title }}]({{ post.url }})
{% endfor %}
