---
title: "<i class='fas fa-newspaper'></i> News Archive"
layout: single
permalink: /news_archive/
sidebar:
  nav: "news"
excerpt: "Pgpool-II news archive"
toc: true
---

{% assign news_items = site.news | sort: "date" | reverse %}

{% for item in news_items %}
## [{{ item.title }}]({{ item.url }})

<p class="news-meta">
Released: {{ item.date | date: "%Y-%m-%d" }}
</p>
---

{% endfor %}
