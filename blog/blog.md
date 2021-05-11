---
layout: post
title: Updates from OpenNews
section: blog
sub-section: index
permalink: /blog/
---

<p class="bodybig">Here's where we share what we're learning, talk about new programs, and post event announcements and application deadlines. If you’d like to get notifications, <a href="https://twitter.com/opennews">Twitter</a> and <a href="http://eepurl.com/czSVTL">our newsletter</a> are great ways to stay up-to-date.</p>

<ul class="bloglist">
  {% for post in site.posts %}
    <li>
      <p class="blogtitle"><a href="{{ post.url }}">{{ post.title }}</a><span class="blogdate">| posted <abbr class="timeago" title="{{ post.date }}">{{ post.date }}</abbr></span></p>
      <p class="excerpt">{{ post.excerpt }}&nbsp;<a href="{{ post.url }}">read more</a></p>
    </li>
  {% endfor %}
</ul>
