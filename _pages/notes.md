---
title: "Notes"
layout: note
permalink: /notes/
toc: false
---

Personal notes — quick references on techniques, tools and concepts I keep coming back to. Pick a topic from the sidebar.

## Categories

{% assign notes_sorted = site.notes | sort: "path" %}
{% assign categories = "" | split: "" %}
{% for n in notes_sorted %}
  {% assign rel = n.path | remove_first: "_notes/" %}
  {% assign parts = rel | split: "/" %}
  {% if parts.size > 1 %}
    {% assign cat = parts[0] %}
  {% else %}
    {% assign cat = "general" %}
  {% endif %}
  {% unless categories contains cat %}
    {% assign categories = categories | push: cat %}
  {% endunless %}
{% endfor %}

<ul>
{% for cat in categories %}
  {% assign cat_label = cat | replace: "-", " " | replace: "_", " " | capitalize %}
  {% assign count = 0 %}
  {% for n in notes_sorted %}
    {% assign rel = n.path | remove_first: "_notes/" %}
    {% assign parts = rel | split: "/" %}
    {% if parts.size > 1 %}{% assign ncat = parts[0] %}{% else %}{% assign ncat = "general" %}{% endif %}
    {% if ncat == cat %}{% assign count = count | plus: 1 %}{% endif %}
  {% endfor %}
  <li><strong>{{ cat_label }}</strong> — {{ count }} note{% if count != 1 %}s{% endif %}</li>
{% endfor %}
</ul>
