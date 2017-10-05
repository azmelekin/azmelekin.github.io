---
layout: page
title: Archive
---
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
       <span class="post-date">{{ post.date | date_to_string }}</span>
    </li>
  {% endfor %}
</ul>