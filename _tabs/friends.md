---
icon: fas fa-user-group
order: 5
title: 友情链接
---

一路同行者，常来看看。欢迎交换友链，联系方式见[「关于」]({{ '/about/' | relative_url }})页。
{% assign friends = site.data.friends %}
{%- if friends.size > 0 %}
<ul class="friend-links">
{%- for friend in friends %}
  <li class="friend-item">
    <div class="friend-avatar"><a href="{{ friend.link }}" title="{{ friend.name | escape }}" target="_blank" rel="noopener noreferrer"><img src="{{ friend.avatar }}" alt="{{ friend.name | escape }}"></a></div>
    <div class="friend-intro">
      <a class="friend-name" href="{{ friend.link }}" target="_blank" rel="noopener noreferrer">{{ friend.name | escape }}</a>
      <p class="friend-desc">{{ friend.desc | escape }}</p>
    </div>
  </li>
{%- endfor %}
</ul>
{%- else %}
暂无友链。
{%- endif %}
