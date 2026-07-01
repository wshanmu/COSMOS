---
layout: page
title: Staff
description: Course staff and learning assistants.
nav_order: 7
---

# Staff

## Instructors

{% assign omid = site.staffers | where: 'slug', 'omid' | first %}
{{ omid }}

{% assign baharan = site.staffers | where: 'slug', 'baharan' | first %}
{{ baharan }}

## Teaching Assistants

{% assign shanmu = site.staffers | where: 'slug', 'shanmu' | first %}
{{ shanmu }}

{% assign jianhao = site.staffers | where: 'slug', 'jianhao' | first %}
{{ jianhao }}

## Learning Assistant

{% assign eric = site.staffers | where: 'slug', 'eric' | first %}
{{ eric }}
