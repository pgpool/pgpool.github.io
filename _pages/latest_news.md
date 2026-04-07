---
title: "<i class='fas fa-newspaper'></i> Latest News"
layout: single
permalink: /latest_news/
sidebar:
  nav: "news"
excerpt: "Latest News"
toc: true
---

## Latest News

{% assign latest = site.news | sort: "date" | reverse | first %}

## [{{ latest.title }}]({{ latest.url }})

{{ latest.content }}

[View All News <i class="fas fa-arrow-right"></i>](/news_archive/){: .btn .btn--primary}
