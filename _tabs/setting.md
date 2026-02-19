---
layout: page
title: Setting
icon: fas fa-gear
order: 2
---
> **알고리즘 동작을 위한 환경설정과 관련된 포스팅 모음**

---

{% assign posts = site.categories.Setting %}
{% for post in posts %}
  <article class="card-wrapper card mb-3">
    <a href="{{ post.url | relative_url }}" class="post-preview row g-0">
      <div class="col-12">
        <div class="card-body">
          <h2 class="card-title my-2">{{ post.title }}</h2>
          <div class="card-text content mt-0 mb-2">
            <p>{{ post.content | strip_html | truncate: 120 }}</p>
          </div>
          <div class="post-meta">
            <i class="far fa-calendar fa-fw me-1"></i>
            {{ post.date | date: "%Y-%m-%d" }}
          </div>
        </div>
      </div>
    </a>
  </article>
{% endfor %}
