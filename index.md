---
layout: content
---

Hi! I'm Martin Larralde, a.k.a @althonos on all things digital, and this is my
attempt at setting up a hacker blog. I'm currently a graduate student in
Bioinformatics and Biostatistics at the École Normale Supérieure Paris-Saclay.

I'm interested in embedded systems, low-level programming, semantic web,
ontologies. I'm currently having fun hacking my Nintendo Switch. Otherwise,
you can find me at a post-punk concert or a new-wave nightclub in the Bay Area.

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

## DevOps and automation

<ul class="posts">
    {% for post in site.categories.devops %}
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
