---
layout: default
title: Archive
---

# Archive

Goongoguma의 블로그 <br />
(오역이 있는 경우, wogus7an@gmail.com으로 알려주시면 감사하겠습니다.)

{% assign postsByYearMonth = site.posts | group_by_exp: "post", "post.date | date: '%B %Y'" %}
{% for yearMonth in postsByYearMonth %}
  <h2>{{ yearMonth.name }}</h2>
  <ul>
    {% for post in yearMonth.items %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}
