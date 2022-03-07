---
layout: page
title: "相册"
subtitle: ""
permalink: /gallery/
gallery_path: "assets/gallery"
tags: [Page]
---

<!-- This is a photo gallery made from the static files in the `assets/img/pexels` folder.
#I wanted to create automatically a simple gallery from a folder without having to create a markdown page as you would for the portfolio.
-->

{% include gallery.html gallery_path=page.gallery_path %}