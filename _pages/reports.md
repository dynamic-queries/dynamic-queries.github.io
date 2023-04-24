---
layout: page
permalink: /reports/
title: Technical Reports
description: A list of technical reports of my formal projects.
years: [2017,2018,2019,2020,2021,2022,2023]
nav: true
nav_order: 2
---
<!-- _pages/publications.md -->
<div class="publications">

{%- for y in page.years %}
  <h2 class="year">{{y}}</h2>
  {% bibliography -f papers -q @*[year={{y}}]* %}
{% endfor %}

</div>