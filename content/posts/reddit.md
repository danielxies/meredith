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

# Yes, they can.

  Here is a high level overview of how I did it.

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
	
comment_to_use: str = highest_score(score_map.values())
```

Now that I had the comment that was most interesting I passed it through a chain of functions that would do everything from creating the text to speech to upload it to youtube. This is what the function chain looked like:

```python
def main(url, title):

	submission = start(url)
	location = f"videos/{submission.title}/{submission.title}_final.mp4"
	output_location = f"videos/{submission.title}/subtitled.mp4"
	create_title_voice_over_tiktok(submission.title, submission.title)
	add_text_to_image(submission.title)
	resize_image_width(f"videos/{submission.title}/{submission.title}.png",
				       f"videos/{submission.title}/{submission.title}.png")
	create_movie_with_background_na(submission.title)
	subtitled_location = subtitles(location, output_location)
	link = youtube(subtitled_location, f"{title}", "reddit", 10, "reddit, funny,                    shorts", "public", False)

return location, link

# CALLING THE FUNCTION
# CALLING THE FUNCTION
# CALLING THE FUNCTION

main("https://www.reddit.com/r/AskReddit/comments/1c7vpsu/what_viral_video_is_fake_but_people_think_its_real/", "#shorts")

```

I'll break down what each part does. The scoring part was the only complex part of this, the rest is pretty easy to understand.

```python
start(url)
```
Takes the url of the reddit post, grabs the comments, and picks a comment to use. It uses the scoring system I explained above. It also creates the text to speech and stores it in the directory structure I made.

```python
location = f"videos/{submission.title}/{submission.title}_final.mp4"
output_location = f"videos/{submission.title}/subtitled.mp4"
```
There are just locations for where the generated videos would be stored. I predefined these so the function arguments wouldn't be super long.

```python
create_title_voice_over_tiktok(submission.title, submission.title)
```
Creates the voice over for the title of the post. The title is usually a question, and is asked first thing in the video. This definitely could be removed by adding the functionality to the `start()` function..... lol.

```python
add_text_to_image(submission.title)
```
Originally, I was using a selenium script to manually open Firefox, navigate to the reddit post, and take a screenshot of the `<h1>` tag as that was a title. However, if I want to host this in the cloud, I don't want it to rely on any external browsers. To fix this, I created a template...

And I could simply add the `title text` on top of it and use that. The `add_text_to_image()` function takes this template and slaps a title onto it, and stores it in the proper directory.

```python
resize_image_width(f"videos/{submission.title}/{submission.title}.png",
				       f"videos/{submission.title}/{submission.title}.png")
```
Reels, tiktok, youtube shorts are all videos with `9x16` aspect ratios, so this just crops the image to fit in my background footage.

```python
create_movie_with_background_na(submission.title)
```
Now that I have all the pieces I need to create a video, this function strings everything together in the order that I defined. 

```python
subtitled_location = subtitles(location, output_location)
```
This is something that is quite interesting. Subtitling videos is actually quite expensive... websites charge upwards of **30$** a month for subtitling services. However, I found a workaround using OpenAI's whisper API. This function just adds subtitles and returns the filepath to the video.

```python
link = youtube(subtitled_location, f"{title}", "reddit", 10, "reddit, funny,                    shorts", "public", False)
```
Lastly, the video is uploaded to youtube. This is done by using google cloud's youtube data API, which limits uploads to 6 per day. 

```python
return location, link
```
Once the video is uploaded, it returns a link to the newly uploaded short and the file location. The chain is now done, and a watchable, subtitled, and entertaining video is now on my channel. 

Compared to editing videos manually using editing software and paying for subtitling, I think this project was pretty cool to show how much can get done by simply knowing which libraries to use. If people made it this far, here are some libraries that are essential.

- `moviepy`
- `whisper`
- `opencv`
- `PIL`
- `numpy`

if you add `@mer49` on discord... you might get the source code.



