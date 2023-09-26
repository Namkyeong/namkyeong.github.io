---
layout: page
permalink: /publications/
title: Publications
description: All the publications that I have done or collaborated with.
sections:
  - bibquery: "@inproceedings"
    text: "International Conferences"
  - bibquery: "@article"
    text: "International Journals"
  - bibquery: "@workshops"
    text: "Workshop Papers"
years: [2023, 2022]
nav: true
---
<!-- _pages/publications.md -->

<div class="publications">

{% for section in page.sections %}

  <a id="{{section.text}}"></a>
  <p class="bibtitle">{{section.text}}</p>

  {%- for y in page.years %}
    <h2 class="year">{{y}}</h2>
    {%- bibliography -f papers -q {{section.bibquery}}[year={{y}}] -%}
  {% endfor %}

{% endfor %}

</div>