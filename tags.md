---
layout: page
title: Tags
permalink: /tags/
---

<ul class="tag-index">
{% for t in site.data.tags %}
  {% assign slug = t[0] %}
  {% assign meta = t[1] %}
  {% assign posts = site.tags[meta.name] %}
  {% if posts %}
  <li>
    <a href="{{ site.baseurl }}/tag/{{ slug }}/">{{ meta.name }}</a> ({{ posts | size }})
    <br>{{ meta.description }}
  </li>
  {% endif %}
{% endfor %}
</ul>
