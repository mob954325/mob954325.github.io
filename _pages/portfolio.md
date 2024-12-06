---
title: Portfolio
layout: collection
permalink: /portfolio/
collection: portfolio # 추가하면 콜렉션 내용이 페이지에 나옴
entries_layout: grid
classes: wide
---

포트폴리오 개발 모음 페이지

<!-- # 2024

<details>
<summary> 2024년 포트폴리오 모음 </summary>

{% for portfolio in site.portfolio %}
  <h2>
    <a href="{{ portfolio.url }}">
      {{ portfolio.title }}
    </a>
  </h2>
  <p>{{ portfolio.content | markdownify }}</p>
{% endfor %}
</details> -->