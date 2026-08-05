---
layout: page
title: FailClosed
permalink: /
subtitle:
css: /assets/css/homepage.css
full-width: true
---

<div class="failclosed-intro">
  <h2>Welcome to FailClosed</h2>
  <p>We are dedicated to cybersecurity, automation, and AI-assisted solutions across various fields, including a growing library of blog posts, CTF writeups, homelab guides, and projects, plus a suite of <a href="/software">free custom GPTs and other tools</a> designed specifically for IT and cybersecurity professionals.</p>
  <p>Browse our <a href="https://failclosed.printify.me/">merch</a> or follow along on <a href="https://x.com/failclosed">X</a>. If you find our content valuable, please support us via <a href="https://ko-fi.com/failclosed">Ko-fi</a>.</p>
</div>

<style>
.failclosed-intro {
  max-width: 700px;
  margin: 0 auto 40px;
  padding: 0 20px;
  text-align: center;
}
.failclosed-intro h2 {
  margin-top: 0;
}
.failclosed-intro p {
  color: #555;
}
</style>

<div class="homepage-grid">

  {%- assign categories_to_show = 
      "blog:Blog,labs:Labs,ctfwriteups:CTF Writeups,homelab:Homelab,projects:Projects" 
      | split: "," -%}

  {%- for pair in categories_to_show -%}
    {%- assign parts = pair | split: ":" -%}
    {%- assign cat_key = parts[0] -%}
    {%- assign cat_display = parts[1] -%}

    {%- assign cat_posts = site.categories[cat_key] | default: empty -%}
    {%- assign cat_count = cat_posts | size -%}

    {%- assign expected_image = "/assets/img/homepage/" 
        | append: cat_key 
        | append: ".png" -%}

    {%- assign cat_image = nil -%}

    {%- for f in site.static_files -%}
      {%- if f.path == expected_image -%}
        {%- assign cat_image = f.path -%}
        {%- break -%}
      {%- endif -%}
    {%- endfor -%}

    <a href="{{ '/' | append: cat_key | append: '/' | relative_url }}" class="homepage-tile">

      <div class="homepage-thumb">
        {% if cat_image %}
          <img src="{{ cat_image | relative_url }}" alt="{{ cat_display }}">
        {% else %}
          <div class="homepage-placeholder">No Image</div>
        {% endif %}
      </div>

      <div class="homepage-title">{{ cat_display }}</div>
      <div class="homepage-count">{{ cat_count }} posts</div>

    </a>

  {%- endfor -%}

</div>

<style>
.homepage-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  justify-content: center;
  margin: 20px 0;
}

.homepage-tile {
  display: block;
  width: 250px;
  border-radius: 10px;
  overflow: hidden;
  text-decoration: none;
  color: inherit;
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
  transition: transform 0.2s, box-shadow 0.2s;
}

.homepage-tile:hover {
  transform: scale(1.05);
  box-shadow: 0 8px 16px rgba(0,0,0,0.3);
}

.homepage-thumb {
  width: 100%;
  height: 150px;
  background: #eee;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

.homepage-thumb img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.homepage-placeholder {
  font-size: 1em;
  color: #555;
}

.homepage-title {
  padding: 5px 10px;
  text-align: center;
  font-weight: bold;
  background: #f8f8f8;
}

.homepage-count {
  text-align: center;
  font-size: 0.9em;
  color: #666;
  padding-bottom: 10px;
}
</style>