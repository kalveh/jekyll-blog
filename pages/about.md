---
layout: page
title: About
description: A glimpse of the passing
keywords: Blog
comments: false
menu: 关于
permalink: /about/
---

人生到处知何似，应似飞鸿踏雪泥。

泥上偶然留指爪，鸿飞那复计东西。


## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
</ul>

