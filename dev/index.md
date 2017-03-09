---
layout: splash
title: "Dev"
---
{% include JB/setup %}

## 1 - System

<ul class="thumbnails">
  {% assign pages_icons = site.categories.sys %}
  {% include custom/pages_reversed %}
</ul>


## 2 - Kernel

<ul class="thumbnails">
  {% assign pages_icons = site.categories.kernel %}
  {% include custom/pages_reversed %}
</ul>

## 3 - Docker

<ul class="thumbnails">
  {% assign pages_icons = site.categories.docker %}
  {% include custom/pages_reversed %}
</ul>
