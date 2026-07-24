---
layout: page
title: Schedule
description: The weekly event schedule.
nav_order: 6
---

# Typical Weekly Schedule

The detailed day-by-day plan is on the [calendar]({{ site.baseurl }}/calendar/). This page shows the usual daily rhythm.

Weeks 3 and 4 use a modified project/lab schedule. Use the [calendar]({{ site.baseurl }}/calendar/) for the exact TA coverage, project blocks, poster deadline, and August 1 demo/presentation location.

{% for schedule in site.schedules %}
{{ schedule }}
{% endfor %}
