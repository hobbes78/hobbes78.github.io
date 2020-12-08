---
layout: home
title: "Welcome to hobbes78's blog"
---
According to Google Trends, blogs have been steadily declining since 2009, with all the social networks that have been popping up. Yet, I still believe they're the unbeatable medium for expressing a mix of text, images and video, with way more freedom. So welcome, don't hesitate to chime in with comments and questions and I hope you leave wiser than you came!

## Posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
