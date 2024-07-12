---
layout: page
permalink: /publications/
title: Publications
description: All the publications that I have done or collaborated with. (†:Equal contribution)
sections:
  - bibquery: "@inproceedings"
    text: "International Conferences"
    years: [2023, 2022]
  - bibquery: "@article"
    text: "International Journals"
    years: [2023, 2022]
  - bibquery: "@workshops"
    text: "Workshop Papers"
    years: [2024, 2023]
nav: true
---
<!-- _pages/publications.md -->

<div class="publications">

{% for section in page.sections %}

  <a id="{{section.text}}"></a>
  <p class="bibtitle">{{section.text}}</p>

  {%- for y in section.years %}
    <h2 class="year">{{y}}</h2>
    {%- bibliography -f papers -q {{section.bibquery}}[year={{y}}] -%}
  {% endfor %}

{% endfor %}

</div>