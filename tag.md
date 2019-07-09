---
layout: blog
title: "Tags"
permalink: /tags/
---

<ul class="tag-box inline">
{% assign list = site.tags | sort %}
    {% for tag in list %}
        <li>
            <a href="#{{ tag[0] }}">
                {{ tag[0] }}
            </a>
            <span>({{ tag[1].size }})</span>
        </li>
    {% endfor %}
{% assign list = nil %}
</ul>

{% assign taglist = site.tags | sort %}
{% for tag in taglist %}
 <h2 id="{{ tag[0] }}">{{ tag[0] }}</h2>
 <ul class="post-list">
  {% assign list = tag[1] %}  
  {% for post in list %}
   <li>
   <a href="{{ post.url }}">{{ post.title }}</a>
   </li>
  {% endfor %}
  {% assign pages_list = nil %}
  {% assign group = nil %}
 </ul>
{% endfor %}
{% assign taglist = nil %}
