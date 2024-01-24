---
title: Ejercicios de minería de conocimientos de Azure
permalink: index.html
layout: home
---

# Ejercicios de minería de conocimientos de Azure

Los siguientes ejercicios están diseñados como apoyo para los módulos de Microsoft Learn.

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
