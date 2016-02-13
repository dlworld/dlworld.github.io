---
layout: page
title: 分类列表
#permalink: /archive/
#banner_image: sample-banner-image-3.jpg
---

<div>
  {% for category in site.categories %}
  <h2>{{ category | first }}</h2>  
  <ul class="arc-list">
    {% for post in category.last %}
    <li><a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}
</div>
