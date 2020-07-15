---
layout: catalog
title: 标签
description: 按标签归档
keywords: 标签
comments: false
menu: 标签
permalink: /tags/
---

<section class="container posts-content">
    {% assign sorted_tags = site.tags | sort %}
    {% for tag in sorted_tags %}
        <h3 id="{{ tag[0] }}">{{ tag | first }}</h3>
        <ol class="posts-list">
            {% for post in tag.last %}
                <li class="posts-list-item">
                    <span class="posts-list-meta">{{ post.date | date:"%Y-%m-%d" }}</span>
                    <a class="posts-list-name" href="{{ site.url }}{{ post.url }}">{{ post.title }}</a>
                </li>
            {% endfor %}
        </ol>
    {% endfor %}
</section>
<!-- /section.content -->
