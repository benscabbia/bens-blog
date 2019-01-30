---
layout: page
title: Categories 
---

{% for category in site.categories %}
  <h3>{{ category[0] }}</h3>
  <ul>
    {% for post in category[1] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}

---

# Tags

{% assign postsSortedByTag = site.tags | sort %}
{% for postContent in postsSortedByTag %}
  {% assign tag = postContent | first %}
  {% assign posts = postContent | last %}

  <div id="{{ tag }}">
    {% assign numPosts = posts | size %}
    {% if numPosts == 1 %}
      <p>{{ numPosts }} post containing tag <b>{{ tag }}</b></p>
    {% else %}
      <p>{{ numPosts }} posts containing tag <b>{{ tag }}</b></p>
    {% endif %}
    <ul class="blog-list">
      {% for post in posts %}
        {% if post.tags contains tag %}
          <li>
            <span class="blog-item-date">{{ post.date | date: "%d %b %Y" }}</span>
            <a href="{{ post.url }}">{{ post.title }}</a>
          </li>
        {% endif %}
      {% endfor %}
    </ul>
  </div>
{% endfor %}
