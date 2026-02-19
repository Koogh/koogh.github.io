---
layout: page
title: SLAM
icon: fas fa-map-marked-alt
order: 3
---

{% assign posts = site.categories['SLAM'] %}
{% for post in posts %}
  <article class="card-wrapper card">
    <a href="{{ post.url | relative_url }}" class="post-preview row g-0 flex-md-row-reverse">
      <div class="col-12">
        <div class="card-body d-flex flex-column">
          <h1 class="card-title my-2 mt-md-0">{{ post.title }}</h1>
          <div class="card-text content mt-0 mb-3">
            <p>{{ post.content | strip_html | truncate: 140 }}</p>
          </div>
          <div class="post-meta flex-grow-1 d-flex align-items-end">
            <div class="me-auto">
              <i class="far fa-calendar fa-fw me-1"></i>
              {{ post.date | date: "%Y-%m-%d" }}
            </div>
          </div>
        </div>
      </div>
    </a>
  </article>
{% endfor %}
