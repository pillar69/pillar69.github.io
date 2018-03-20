---
layout: default
---

{% for post in site.posts %}
*    [{{ post.excerpt | remove: '<p>' | remove: '</p>' remove: '<h1>' | remove: '</h1>' }}]({{ post.url }})
{% endfor %}
