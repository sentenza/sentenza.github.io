---
layout: index
title: Alfredo Torre 
tagline: aka sentenza
---
{% include JB/setup %}


<!-- This is not shown on the index template page! -->

Here's a sample "posts list".

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
