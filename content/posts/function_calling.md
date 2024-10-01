---
title: "introduction to openai function calling"
date: false
weight: 3
# aliases: ["/first"]
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
tags: [machine learning, python, tutorials]
---

# Key Topics

- [**OpenAI Function Calls**](https://platform.openai.com/docs/guides/function-calling)
- **State Handling**
- **API Integration**

## Abstract

```a title="console"
q: what is the uuid of the player merrydith?
```
`Question: How can my code respond to human like speech input?`


This project is a demonstration on how a large language model can access information outside of the data set it was trained on. OpenAI has added the capability to create functions (ex. accessing mojang api) and call respective functions when the language model sees fit.

**More importantly**, it shows how a language model can process natural language into a format that API's and computers can recognize. It *allows* us humans to communicate as a **human** to our code. Think about those AI chatbots on shopping stores. They 100% use the idea discussed in this article.


## Goal

Say I type a sentence asking, `"what is the id of this player?"` to a python script. It would store the input in some variable, and try to pass it into an API that gets the id. However, the API would recieve the entire sentence, which would result in an error because the sentence is **NOT** a valid username.

If only there was a way to extract ONLY the username out of the sentence, and then pass it into the API...

There is! OpenAI and most high end language models offer a strategy called `function calling`, which is better explained [here.](https://platform.openai.com/docs/guides/function-calling) This article is just an example you can try once you get an understanding of what function calling is.

- `queued` The request is queued for processing. This status indicates that the process has started but has not yet begun extracting the relevant data. 
- `in_progress` The processing of the request is now in progress. At this stage, the script is actively working to extract the username from the provided sentence.
- `requires_action` This status typically indicates that additional input or confirmation may be needed; however, in this automated process, it proceeds swiftly to the next step.
- `completed` The process is completed, and the UUID has been successfully retrieved.

### Here is what the complete chain of events looks like:

```a title="console"
q: what is the id of the player username merrydith 
```

```a title="status 1: queued"
 "status": "queued",
```

```a title="status 2: in_progress"
"status": "in_progress",
```

```a title="function call"
Function Calling
{'tool_calls': [{'id': 'merrydith', 'function': {'arguments': '{\n  "username": "merrydith"\n}', 'name': 'get_uuid_from_username'}, 'type': 'function'}]}
Submitting outputs back to the Assistant...
```

```a title="status 3: requires_action"
"status": "requires_action",
```

```a title="status 4: in_progress"
"status": "in_progress",
```

```a title="status 5: completed"
"status": "completed",
```

```a title="output"
Assistant: The id for "merrydith" is 4e0b33ffc796431aa99253626e33d4a6.
```


















## Code

```python title="python 3.11"
from openai import OpenAI
import time
import os
from dotenv import load_dotenv
import requests

load_dotenv()
openai = os.getenv('OPENAI_API_KEY')
hypixel = os.getenv('HYPIXEL_API_KEY')

prompt = input("q: ")

#------------FUNCTIONS-----------------#

def get_uuid_from_username(username: str) -> str:
    url = "https://api.mojang.com/users/profiles/minecraft/{username}"
    response = requests.get(url.format(username=username))
    if response.status_code == 200:
        data = response.json()
        uuid = data.get("id")
        return uuid
    else:
        return "error"


#------------TOOLS and CLIENT-----------------#

tools_list = [{
    
        "type": "function",
        "function": {

            "name": "get_uuid_from_username",
            "description": "Retrieve the UUID given a minecraft username.",
            "parameters": {
                "type": "object",
                "properties": {
                    "username": {
                        "type": "string",
                        "description": "The minecraft username."
                    }
                },
                "required": ["username"]
            }
        }
    
}]


client = OpenAI(api_key=openai)

#create assistant
assistant = client.beta.assistants.create(
    name="daniels assistant",
    instructions="You are a personal assistant that caters to any task.",
    tools=tools_list,
    model="gpt-3.5-turbo-0613",
)

#create thread
thread = client.beta.threads.create()

#add msg to thread
message = client.beta.threads.messages.create(
    thread_id=thread.id,
    role="user",
    content=prompt
)

#run the assistant
run = client.beta.threads.runs.create(
    thread_id=thread.id,
    assistant_id=assistant.id,
    instructions="Please address the user as Johnny Appleseed"
)

print(run.model_dump_json(indent=4))

while True:

    time.sleep(1)

    #retrieve the run status
    run_status = client.beta.threads.runs.retrieve(
        thread_id=thread.id,
        run_id=run.id
    )
    print(run_status.model_dump_json(indent=4))

    # If run is completed, get messages
    if run_status.status == 'completed':
        messages = client.beta.threads.messages.list(
            thread_id=thread.id
        )

        # Loop through messages and print content based on role
        for msg in messages.data:
            role = msg.role
            content = msg.content[0].text.value
            print(f"{role.capitalize()}: {content}")

        break

    elif run_status.status == 'requires_action':
        print("Function Calling")
        required_actions = run_status.required_action.submit_tool_outputs.model_dump()
        print(required_actions)
        tool_outputs = []
        import json
        for action in required_actions["tool_calls"]:
            func_name = action['function']['name']
            arguments = json.loads(action['function']['arguments'])

            if func_name == "get_uuid_from_username":
                output = get_uuid(username=arguments['username'])
                tool_outputs.append({
                    "tool_call_id": action['id'],
                    "output": output
                })

            else:
                raise ValueError(f"Unknown function: {func_name}")
            
        print("Submitting outputs back to the Assistant...")
        client.beta.threads.runs.submit_tool_outputs(
            thread_id=thread.id,
            run_id=run.id,
            tool_outputs=tool_outputs
        )
    else:
        print("Waiting for the Assistant to process...")
        time.sleep(1)

```


