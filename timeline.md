---
layout: blog
title: "Timeline"
permalink: /timeline/
---

<ul class="posts">
    {% for post in site.categories.blog %}
        <li>
            <span class="post-date">{{ post.date | date: "%b %d, %Y" }}</span>
            ::
            <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
            <br>
            @ {
            {% assign tag = post.tags | sort %}
            {% for t in tag %}<span><a href="{{ 'tags' | relative_url }}/#{{ t }}" class="reserved">{{ t }}</a>{% if forloop.last != true %}, {% endif %}</span>{% endfor %}
            {% assign tag = nil %}
            }
        </li>
    {% endfor %}
</ul>
