---
layout: page
title: Recent posts
tagline:
---
{% include JB/setup %}

{% for post in site.posts %}

   <!-- here add you post markup -->
   <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
   <p class="author">
    <span class="date">{{post.date}}</span>
  </p>
  <div class="content">
    {{ post.content }}
  </div>
{% endfor %}

