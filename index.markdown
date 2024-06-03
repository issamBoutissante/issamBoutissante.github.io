---
layout: home
title: Issam Boutissante's Blog
---

![Issam Boutissante's Banner](assets/banner.png)

<div class="blog-posts">
  {% for post in site.posts %}
  <div class="post-preview">
    <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
    <p class="post-meta">{{ post.date | date: "%B %d, %Y" }}</p>
    <p>{{ post.excerpt }}</p>
    <a href="{{ post.url }}" class="read-more">Read More</a>
  </div>
  <hr>
  {% endfor %}
</div>

<style>
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

  .post-preview p {
    font-size: 1em;
    line-height: 1.6;
  }

  .post-meta {
    color: #666;
    font-style: italic;
    margin-bottom: 10px;
  }

  .read-more {
    display: inline-block;
    margin-top: 10px;
    padding: 5px 10px;
    background-color: #007BFF;
    color: #fff;
    text-decoration: none;
    border-radius: 5px;
    transition: background-color 0.3s ease;
  }

  .read-more:hover {
    background-color: #0056b3;
  }
</style>
