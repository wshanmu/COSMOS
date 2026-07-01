---
layout: page
title: Calendar
description: Listing of course modules and topics.
nav_order: 2
---

# Calendar

Lecture meets 9:00-11:30 AM. Lab meets 1:00-5:00 PM unless a special event is listed.

{% for module in site.modules %}
{{ module }}
{% endfor %}
