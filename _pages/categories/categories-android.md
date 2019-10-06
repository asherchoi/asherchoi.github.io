---
title: "Algorithm Posts"
permalink: /categories/android/
layout: archive
author_profile: true
---

{% assign posts = site.categories.Android | sort:'date' | reverse %}

{% for post in posts %}
    {% include archive-single.html %}
{% endfor %}
