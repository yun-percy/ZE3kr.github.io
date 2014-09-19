---
layout: page
title: 2013年分类
notpost: yes
---

<ul>
{% for cat in site.categories %}
  <b id="{{ cat[0] | cgi_escape }}">{{ cat[0] }}</b>
{% for post in cat[1] %}
{% capture y %}{{ post.date | date:"%Y"}}{% endcapture %}
{% capture year %}2013{% endcapture %}
	{% if y != year %}
	{% else %}
  <li>
		<a class="post-date" href="{{ post.url }}" title=""><b class="gray">{{ post.date | date: "%-m月%-d日" }}</b>
		<span>{{ post.title }}</span></a>
  </li>
	{% endif %}
{% endfor %}
{% endfor %}
</ul>