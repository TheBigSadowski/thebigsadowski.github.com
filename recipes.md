---
layout: plain
---

# Cooking
Tips, tricks, tools, and recipes that I've been keeping track of to catalogue and share food I've made.

# Recipes
{% for recipe in site.recipes %}
<h2><a href="{{ recipe.url }}">{{ recipe.title }}</a></h2>
<p>{{ recipe.description }}</p>
{% endfor %}
