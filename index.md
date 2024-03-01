---
title: A Geekier Blog
---

A place for some of my tech writeups. For additional details, see the [README.md](https://github.com/mmartial/geekier-blog/blob/main/README.md) on GH.

## Index of Posts
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>


