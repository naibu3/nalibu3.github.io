---
layout: page
permalink: /categories/
title: Categories
---

<div id="archives" class="categories-accordion">
  {% for category in site.categories %}
    <details class="archive-group" id="{{ category | first | slugize }}" {% if forloop.first %}open{% endif %}>
      {% capture category_name %}{{ category | first }}{% endcapture %}
      <summary class="uml-header">
        <span class="category-name">{{ category_name }}</span>
        <span class="category-count">{{ site.categories[category_name] | size }}</span>
      </summary>
      <div class="posts-list" id="posts-{{ category_name | slugize }}">
        {% for post in site.categories[category_name] %}
          <article class="archive-item">
            <a href="{{ site.baseurl }}{{ post.url }}">
              {% if post.title and post.title != "" %}
                {{ post.title }}
              {% else %}
                {{ post.excerpt | strip_html }}
              {% endif %}
            </a>
          </article>
        {% endfor %}
      </div>
    </details>
  {% endfor %}
</div>


