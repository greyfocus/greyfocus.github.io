---
layout: page
title: Archive
---

{% for post in site.posts %}
  {% assign currentdate = post.date | date: "%Y" %}
  {% if currentdate != date %}
    {% unless forloop.first %}</ul>{% endunless %}
<h2>{{ currentdate }}</h2>
<ul>
    {% assign date = currentdate %}
  {% endif %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% if forloop.last %}
</ul>
  {% endif %}  
{% endfor %}
