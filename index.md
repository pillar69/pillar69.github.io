---
layout: default
---

{% for post in site.posts %}
*    [{{ post.excerpt }}]({{ post.url }})
{% endfor %}
