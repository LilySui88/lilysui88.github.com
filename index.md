---
layout: page
title: Welcome to lily's  World!
tagline: Good Good Study,Day Day Up
---
{% include JB/setup %}

## Coding
    Just keep on coding

## All My Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

