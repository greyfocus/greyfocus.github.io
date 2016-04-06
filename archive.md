---
layout: page
title: Archive
---

## Blog Posts

{% for post in site.posts %}
  {% assign currentdate = post.date | date: "%Y" %}
  {% if currentdate != date %}
    {% unless forloop.first %}</ul>{% endunless %}
    <h2>{{ currentdate }}</h2>
    <ul>
    {% assign date = currentdate %}
  {% endif %}
  <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% if forloop.last %}</ul>{% endif %}
  * {{post.date | date_to_string}} &raquo; [ {{post.title}} ] ( {{post.url}} )
  {% endif %}
{% endfor %}
