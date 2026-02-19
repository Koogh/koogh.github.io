---
layout: page
title: SLAM
icon: fas fa-map-marked-alt
order: 3
---
> **SLAM 알고리즘 공부 포스팅**
---
{% assign posts = site.categories.SLAM %}
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
