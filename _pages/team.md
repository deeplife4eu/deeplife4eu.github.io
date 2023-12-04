---
title: "DeepLife - Team"
layout: gridlay
excerpt: ""
sitemap: false
permalink: /team/
---

### Contributors

{% for article in site.data.team_members %}
- <p> {{ article.name }} ({{ article.university }}) <br> contact: {{ article.email}}</p>
{% endfor %}

### Funding

<br>
<br>