---
---
<ul class="list-group">
  {% for page in site.pages %}
    {% if page.url != "/" %}
        <li class="list-group-item">
          <a href="{{ page.url | relative_url }}">{{ page.title }}</a>
        </li>
    {% endif %}
  {% endfor %}
</ul>
