---
title: "automating youtube shorts with python"
date: false
weight: 1
aliases: ["/automation"]
tags: ["python"]
author: "daniel"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: ""
canonicalURL: "https://canonical.url/to/page"
disableHLJS: false # to disable highlightjs
disableShare: true
disableHLJS: false
hideSummary: true
searchHidden: false
ShowReadingTime: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page


categories: [python]
tags: [python]
---

# Abstract

Youtube shorts, instagram reels, tiktok– these are all short videos. There is one niche in particular, where entertaining background footage is combined with text to speech storytelling that pulls quite a bit of traction and viewership.

Once you look at these videos a little more, you realize how simple they are. Thus, can they be automated?

## Yes, they can.

Here is a high level overview to how I started with nothing, and eventually built a discord bot and a frontend that **could do every part of the process**, from `finding a story` to `uploading` it to social media platforms.