---
layout: page
title: Javascript
excerpt: "An archive of articles sorted by date."
---

<ul class="post-list">
{% for post in site.categories.java %} 
  <li><article><a href="{{ site.url }}{{ post.url }}">{{ post.title }} <span class="entry-date"><time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%B %d, %Y" }}</time></span></a></article></li>
{% endfor %}
</ul>
