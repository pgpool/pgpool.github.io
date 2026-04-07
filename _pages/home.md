---
title: "Pgpool-II"
layout: splash
permalink: /
hidden: true
header:
  overlay_color: "#5e616c"
  actions:
    - label: "<i class='fas fa-download'></i> Download Now"
      url: "/download/"
    - label: "<i class='fas fa-file-lines'></i> Documetation"
      url: "/documentation/"
excerpt: >
  Feature-rich, open-source middleware for PostgreSQL connection pooling, load balancing and high availability.<br />
  <small><a href="/latest_news/">Latest release</a></small>
feature_row:
  - title: "<i class='fas fa-unlock'></i> 100% Open Source"
    alt: "100% free"
    excerpt: "Fully open source, giving you complete control with no vendor lock-in."
    url: "/license/"
    btn_class: "btn--primary"
    btn_label: "Learn more"
  - title: "<i class='fas fa-sliders-h'></i> Feature-rich"
    alt: "feature rich"
    excerpt: "Offers connection pooling, load balancing and high availability."
    url: "/docs/latest/en/html/intro-whatis.html"
    btn_class: "btn--primary"
    btn_label: "See All Features"
  - title: "<i class='fas fa-play'></i> Getting Started"
    alt: "getting Started"
    excerpt: "Set up your high-availability cluster step by step."
    url: "/docs/latest/en/html/example-cluster.html"
    btn_class: "btn--primary"
    btn_label: "Tutorial"      
---

{% include feature_row %}

## Latest News

{% assign latest = site.news | sort: "date" | reverse | first %}

## [{{ latest.title }}]({{ latest.url }})

{{ latest.content }}

[View All News <i class="fas fa-arrow-right"></i>](/news_archive/){: .btn .btn--primary}
