---
author: Sato Seinosuke
title: Landing Page
description: notes and blogs
heading: "一所懸命"
tagline: "信は望む所の實体にして 未だ見ぬ事の證據なり"
content_blocks:
  - _bookshop_name: hero
    heading:
      title: "一所懸命"
      content: |-
        信は望む所の實体にして 未だ見ぬ事の證據なり
      width: 6
    background:
      color: primary
      subtle: true
    illustration:
      image: /img/sunrise.jpg
      ratio: 16x9
    width: 8
    orientation: horizontal
    justify: center

  - _bookshop_name: articles
    heading:
      title: Blogs
      align: start
    background:
      background: body-tertiary
    hide-empty: false
    input:
      section: blogs
      reverse: true
      sort: date
    header-style: none
    more:
      title: More blogs
    cols: 3
    padding: 4
    limit: 6
    icon-style: fa-5x
    footer-style: tags
    orientation: horizontal-sm
    class: border-1 card-emphasize

  - _bookshop_name: articles
    heading:
      title: Notes
      align: start
    background:
      background: body-tertiary
    hide-empty: false
    input:
      section: notes
      reverse: true
      sort: date
    more:
      title: More notes
    cols: 3
    padding: 4
    limit: 6
    icon-style: fa-5x
    header-style: none
    footer-style: tags
    orientation: horizontal-sm
    class: border-1 card-emphasize
---
