---
layout: default
title: My Blog
description: Personal blog about my projects and other stuff I'm interested in
---
<h1>{{ page.title }}</h1>
<ul class="posts">
  {% for post in site.posts %}
    <li class="list-group-item">
      <span>{{ post.created_date | date: "%Y-%m-%d" }}</span>
      <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

{% if jekyll.environment == 'production' and site.google_track_id %}
  {% include analytics.html %}
{% endif %}
