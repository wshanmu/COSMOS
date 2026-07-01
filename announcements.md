---
layout: page
title: Announcements
description: Course announcements.
nav_order: 9
---

# Announcements

{% for announcement in site.announcements reversed %}
{{ announcement }}
{% endfor %}
