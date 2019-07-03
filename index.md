---
layout: content
---

Hi! I'm Martin Larralde, a.k.a @althonos on all things digital, and this is my
attempt at setting up a hacker blog.

## Open-sourcing the PICO-8

<ul class="posts">
    {% for post in site.categories.pico8 %}
        <li>
            <span class="post-date">{{ post.date | date: "%b %d, %Y" }}</span>
            ::
            <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
            <br>
            @ {
            {% assign tag = post.tags | sort %}
            {% for category in tag %}<span><a href="{{ 'categories' | relative_url }}/#{{ category }}" class="reserved">{{ category }}</a>{% if forloop.last != true %}, {% endif %}</span>{% endfor %}
            {% assign tag = nil %}
            }
        </li>
    {% endfor %}
</ul>

## Me

<ul class="posts">
    {% for post in site.categories.life %}
        <li>
            <span class="post-date">{{ post.date | date: "%b %d, %Y" }}</span>
            ::
            <a class="post-link" href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
            <br>
            @ {
            {% assign tag = post.tags | sort %}
            {% for category in tag %}<span><a href="{{ 'categories' | relative_url }}/#{{ category }}" class="reserved">{{ category }}</a>{% if forloop.last != true %}, {% endif %}</span>{% endfor %}
            {% assign tag = nil %}
            }
        </li>
    {% endfor %}
</ul>
