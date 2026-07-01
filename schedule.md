---
layout: page
title: Schedule
description: The weekly event schedule.
nav_order: 6
---

# Typical Weekly Schedule

The detailed day-by-day plan is on the [calendar]({{ site.baseurl }}/calendar/). This page shows the usual daily rhythm.

{% for schedule in site.schedules %}
{{ schedule }}
{% endfor %}
