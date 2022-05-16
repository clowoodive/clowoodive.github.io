---
title: "디자인 패턴"
layout: archive
permalink: categories/design-pattern
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.['디자인 패턴'] %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}