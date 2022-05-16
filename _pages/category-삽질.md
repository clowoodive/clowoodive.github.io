---
title: "삽질"
layout: archive
permalink: categories/삽질
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.['삽질'] %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}