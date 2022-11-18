---
layout: default
title: My Blog
description: Personal blog about my projects and other stuff I'm interested in
---
<h1>{{ page.title }}</h1>
<ul class="posts">
  {% for post in site.posts %}
    <li class="list-group-item">
      <span>{{ post.date | date_to_string }}</span>
      <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
