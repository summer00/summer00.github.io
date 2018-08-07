---
title: "Tags"
---

{% for category in site.categories %}
<h2>{{ category | first }} </h2>

<ul>
    {% for post in category.last %}
        <li><a href="{{ post.url }}">{{ post.title }}</a> {{ post.date | date:"%d/%m/%Y"}}</li>
    {% endfor %}
</ul>

{% endfor %}