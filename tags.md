---
title: "Tags"
---

{% for category in site.categories %}
<h2>{{ category | first }} </h2>

<ul>
    {% for post in category.last %}
        <li> {{ post.date | date:"%Y-%m-%d"}} <a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>

{% endfor %}