---
layout: page
permalink: /publications/
title: Publications
description: A list of posters, peer-reviewed publications and conference proceedings.
years: [2021,2023]
nav: 
nav_order: 2
---
_pages/publications.md
<div class="publications">

<!-- {%- for y in page.years %} -->
  <!-- <h2 class="year">{{y}}</h2> -->
  <!-- {% bibliography -f papers -q @*[year={{y}}]* %} -->
  {% bibliography -f {{ site.scholar.bibliography }} %}
<!-- {% endfor %} -->

</div>
