---
title: Temas
subtitle: Temas del blog
description: 
featured_image: /images/demo/demo-portrait.jpg
---

{% for category in site.categories %}
  <h3>{{ category[0] }}</h3>
  <ul>
    {% for post in category[1] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}