---
title: A Geekier Blog
---

A place to put different writeups I did and use this page as an indexer for it.
Easier than having multiple repositories.

## Index of Posts
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{site.baseurl}}{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>


