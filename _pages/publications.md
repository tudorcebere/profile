---
layout: archive
title: "Publications"
permalink: /publications/
author_profile: true
---

  You can also find my up to date articles on <u><a href="https://scholar.google.com/citations?user=uunZxOUAAAAJ&hl=en">my Google Scholar profile</a>.</u>

{% include base_path %}

{% for post in site.publications reversed %}
  {% include archive-single.html %}
{% endfor %}
