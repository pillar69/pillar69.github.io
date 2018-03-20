---
layout: default
---

{% for post in site.posts %}
*    [{{ post.excerpt | remove: '<p>' | remove: '</p>' }}]({{ post.url }})
{% endfor %}
