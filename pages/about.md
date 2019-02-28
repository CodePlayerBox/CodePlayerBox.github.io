---
layout: page
title: 关于
description: 学编程 玩代码
keywords: CodePlayerBox, Scratch
comments: true
menu: 关于
permalink: /about/
---

> 关于


## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## 技能列表

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}

