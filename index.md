---
title: A Geekier Blog
---

20240518 Update: 
- Content has moved to [https://blg.gkr.one/](https://blg.gkr.one/)
- Having been working on new content in [Notion](https://notion.so) I have decided to move the previous content there as well
  - I used [NoteHpst](https://github.com/velsa/notehost) to link the primary page to a subdomain to keep the dynamic filtering table
  - I am also using [simple.ink](https://simple.ink] to create static pages for viewing.

Hopefully, this content proves useful to you, as they are for me.

"Make Good Tech"  -- M
## Index of Posts
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>


