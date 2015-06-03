---
layout: post-no-feature
title: Scripts
permalink: /scripts/
---
Just scratching my own itch here.

{% if site.scripts %}
{% for script in site.scripts %}
* [{{ script.title }}]({{ script.url | prepend: site.baseurl }})
{% endfor %}	
{%endif%}