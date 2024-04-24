---
title: "automating youtube shorts with python"
date: false
weight: 1
# aliases: ["/automation"]
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

  Youtube shorts, Instagram reels, Tiktok– these are all short videos. There is one niche in particular, where entertaining background footage is combined with text to speech storytelling that pulls quite a bit of traction and viewership.

  Once you look at these videos a little more, you realize how simple they are. Thus, can they be automated?

## Yes, they can.

  Here is a high level overview of how I did it.

![Screenshot](https://cdn.discordapp.com/attachments/975237011290071040/1232546314362028063/Screenshot_2024-04-24_at_12.16.13_AM.png?ex=6629d9b4&is=66288834&hm=b42e3dbedf31f180bd86e6fb7880540f155e0a9226223a5fbc90a3e615eb3df8&)

In the picture above, I create a reddit API instance and pull the top 400 comments from the post that I am interested in. The question is, how did I pick the comment I would use for my text to speech?

Early on, I was simply taking the **top comment** and just using that, but I noticed that the text to speech generated was pretty damn boring most of the time. For example, if the question:

`What is the most dangerous thing you have ever encountered?`

I would get videos where the story would be... pretty boring LOL. Recycling, Trash Cans, Mosquitos... I needed something that someone would stay and listen to. This led me to implement a way to rank each comment based on how interesting the comment is. How can we do this without human intervention? Large Language Models.

What I did was take each comment that fit the criteria I was looking for...

```python
if 500 < len(comment) < 700
	usable_comments.append(comment)
```

And use the `GroqCloud API` to score the comment on how interesting it would be to someone who has to listen to it play as text to speech.

I defined a function called `score()` which returns an integer score for the comment, and called that function on the `usable_comments` array. If you were curious, here is the pseudocode for the score function:

```python
# THIS CODE DOES NOT WORK IT IS JUST FOR THE IDEAS
# THIS CODE DOES NOT WORK IT IS JUST FOR THE IDEAS
# THIS CODE DOES NOT WORK IT IS JUST FOR THE IDEAS

def score(comment: str) -> int:
	
	client = groqclient.init()
	response = completion(
		{"system": "score this comment..."},
		{"content": comment}
	)

	score = response
	return score
```

To store the data, I created a dictionary that had the _comment_ as the key, and the **score** as the value.

```python
score_map = {}

for comment in usable_comments:
	result: int = score(comment)
	score_map.add(comment, result)
```

The comment I would use for my video would be the one with the highest score... obviously. However, I thought I would go a step further 
