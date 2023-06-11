---
layout: page
title: Veritas-Si-ium
permalink: /veritasium/
description: A collection of computational physics projects inspired by Veritasium.
nav: true
nav_order: 5
display_categories: []
horizontal: true
---

[Veritasium](https://www.youtube.com/@veritasium) = ["Veritas"](https://en.wikipedia.org/wiki/Veritas) + "ium", is a made up element in the periodic table of the internet. It is a popular science education channel on YouTube, committed to conveying complex scientific ideas in everyday language without compromising on depth. Here is a collection of in-silico representations of ideas conveyed in Veritasium's videos, hence Veritas-Si-ium :grin:.

<!-- pages/vertasium.md -->
<div class="projects">
{%- if site.enable_project_categories and page.display_categories %}
  <!-- Display categorized vertasium -->
  {%- for category in page.display_categories %}
  <h2 class="category">{{ category }}</h2>
  {%- assign categorized_projects = site.vertasium | where: "category", category -%}
  {%- assign sorted_projects = categorized_projects | sort: "importance" %}
  <!-- Generate cards for each project -->
  {% if page.horizontal -%}
  <div class="container">
    <div class="row row-cols-2">
    {%- for project in sorted_projects -%}
      {% include projects_horizontal.html %}
    {%- endfor %}
    </div>
  </div>
  {%- else -%}
  <div class="grid">
    {%- for project in sorted_projects -%}
      {% include projects.html %}
    {%- endfor %}
  </div>
  {%- endif -%}
  {% endfor %}

{%- else -%}
<!-- Display projects without categories -->
  {%- assign sorted_projects = site.vertasium | sort: "importance" -%}
  <!-- Generate cards for each project -->
  {% if page.horizontal -%}
  <div class="container">
    <div class="row row-cols-2">
    {%- for project in sorted_projects -%}
      {% include projects_horizontal.html %}
    {%- endfor %}
    </div>
  </div>
  {%- else -%}
  <div class="grid">
    {%- for project in sorted_projects -%}
      {% include veritasium.html %}
    {%- endfor %}
  </div>
  {%- endif -%}
{%- endif -%}
</div>
