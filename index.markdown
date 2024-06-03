---
layout: custom_home
title: Issam Boutissante's Blog
---

![Issam Boutissante's Banner](assets/banner.png){: .img-fluid .banner-img}

## Posts

<div class="blog-posts">
  {% for post in site.posts %}
  <div class="post-preview">
    <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
    <p class="post-meta">{{ post.date | date: "%B %d, %Y" }}</p>
  </div>
  <hr>
  {% endfor %}
</div>

<style>
  .banner-img {
    max-height: 300px;
    width: 100%;
    object-fit: cover;
  }

  .blog-posts {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
  }

  .post-preview {
    margin-bottom: 40px;
  }

  .post-preview h2 {
    font-size: 1.5em;
  }

  .post-meta {
    color: #666;
    font-style: italic;
    margin-bottom: 10px;
  }
</style>
