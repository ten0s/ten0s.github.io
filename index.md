---
layout: default
---
<ul class="list-group">
  {% for post in site.posts %}
    <li class="list-group-item">
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
