---
title: "fixing code style with style"
date: false
weight: 2
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
tags: [machine learning, python, openai]
---

# Key Topics

- **OpenAI API**
- **Function Chains**
- **Validation**
- **Prompt Engineering**

# Abstract

In my university computer science course, we have an extensive code standard. We are provided with a linter that is a bash script and identifies most errors, but it doesn't cover every case, such as **embedded constants** and **local variable definitions**. In this project, I will focus on **embedded constants** and cover 1. identifying these, and 2, fixing these.

The strategy I use to solve this issue can be applied to anything similar. To incite thought, think about _something that is redundant_. If you are familiar with large language models, think about something a large language model could help you automate. Anyways here is an example:
```c
Here is a Bad Example of Embedded Constant:

if (temperature > 10)   /* DO NOT USE */
Again, in this case the value means something, and you
should define a constant instead...
#define MAX_TEMPERATURE  (10)
```
## First Approach

I will be referencing the ChatGPT API quite a bit in the following text. If you are not comfortable with terminology, here is the link to the offical [documentation](https://platform.openai.com/docs/api-reference). I am proceeding assuming you know how an API works and what it does.

The first approach I did was to do both steps (identifying and fixing) in **one** step. My API Request looked similar to this.

```python
completion = client.chat.completions.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content":
"""
Code Standard Check: 'IX. DEFENSIVE CODING TECHNIQUE' compliance for C code. This tool inspects C code for adherence to defensive programming practices.

Rules:
- (IX.A) Check or return the function's return value for error conditions.
- (IX.B) Use fclose to close files opened with fopen and set the file pointer to NULL afterwards.
- (IX.C) After using free on a pointer, set it to NULL.
- (IX.D) Perform range checking on function parameters.
- (IX.E) Use the correct typed symbol to represent 0 and NULL.

Violation Examples:

- Rule IX.A Violations (EXTREMELY IMPORTANT):
  FILE *file = fopen("data.txt", "r"); // Missing check for NULL after fopen
  char *data = (char *)malloc(100); // Missing check for NULL after malloc
  size_t result = fwrite(buffer, sizeof(char), 100, file); // Missing check on fwrite return value

- Rule IX.B Violations: (EXTREMELY IMPORTANT)
  FILE *file = fopen("data.txt", "w"); fclose(file); // Missing file pointer set to NULL after fclose
  file = fopen("data.txt", "r"); if (file) { /* ... */ fclose(file); } // Correct use, but missing setting pointer to NULL

- Rule IX.C Violations:
  char *data = (char *)malloc(50); free(data); // Missing set to NULL after free
  int *numbers = (int *)calloc(20, sizeof(int)); free(numbers); // Correct free, but missing set to NULL

- Rule IX.D Violations
  void setTemperature(int temperature) { if (temperature < 0 || temperature > 100) { /* ... */ } } // Missing standardized range checking
  if (age >= 18 && age <= 130) { /* ... */ } // Correct range, but consider using constants for readability

- Rule IX.E Violations:
  int is_done = NULL; // Incorrect symbol, should use 0 for integers
  float value = NULL; // Incorrect symbol, should use 0.0 for reals

Error Reporting Format:

The following are the Error definitions to use: 

- (IX.A) Check Function Return - Ensure return values of functions are checked or returned.
- (IX.B) File Close - Ensure files are closed and pointers are set to NULL.
- (IX.C) Pointer Reset After Free - Ensure pointers are set to NULL after free.
- (IX.D) Parameter Range Checking - Ensure parameters are within expected range.
- (IX.E) Typed Symbol Usage - Ensure the correct symbol is used for 0 and NULL.

- Errors are reported as '[RULE].[LETTER] - Error definition - Error Description, followed by a new line.

[IX.A] - Check Function Return - FILE *file = fopen("data.txt", "r"); - Missing check for NULL after fopen.
[IX.B] - File Close - FILE *file = fopen("data.txt", "w"); fclose(file); - Missing file pointer set to NULL after fclose.
[IX.C] - Pointer Reset After Free - char *data = (char *)malloc(50); free(data); - Missing set to NULL after free.
[IX.D] - Parameter Range Checking - void setTemperature(int temperature) { if (temperature < 0 || temperature > 100) { /* ... */ } } - Missing standardized range checking.
[IX.E] - Typed Symbol Usage - int is_done = NULL; - Incorrect symbol, should use 0 for integers.

Example Response:
The prompt should only have the error codes. No extra text. Alphabetical order.
If no errors are found, the response should be empty.


[IX.A] - Check Function Return - FILE *file = fopen("data.txt", "r"); - Missing check for NULL after fopen.
[IX.A] - Check Function Return - char *data = (char *)malloc(100); - Missing check for NULL after malloc.
[IX.C] - Pointer Reset After Free - int *numbers = (int *)calloc(20, sizeof(int)); free(numbers); - Correct free, but missing set to NULL.

"""
            },
            {"role": "user", "content": text_data}
        ]
    )

  response = completion.choices[0].message.content
```

However, this did not work. Something I quickly learned was the **bigger** the system prompt, the higher chance the language model has to _forget, skip over something, or mess up in general_. Also, I was simply copy and pasting straight from the code standard and passing in vague examples.

I had to separate each step for more accurate results. Reiterating, there are `two` key steps for solving the issue of embedded constants.

### `1. Identify which Constants are wrong.`
### `2. Modify the Code to reflect the changes.`

First, I aimed to do step 1 in its **own request**. This means the language model is ONLY trying to identify constants, it's not trying to fix them as well which allows for better focus on the task.

This is the system prompt I passed in:
```python
"role": "system", "content":
```

```txt
Analyze the provided code snippets and identify any instances
where literal numbers (embedded constants) are used directly
in conditions, return statements, or variable initializations.
Instead of using these literals directly, suggest defining them
as constants at the beginning of the program to improve code
readability and maintainability. Return these values and their
line numbers in a python dictionary in the format of
[constant_value]:[line_number]
```

Once I had access to the constants to check and their line numbers, the second api request was quite straightfoward. I passed in the dictionary and the code, and the language model knew exactly where to look, completely removing the effort for searching.

This proved quite effective in producing accurate results. Here is the full completion request for the second prompt. 

```txt
messages = [
  {"role": "system", "content":
  You will be given a dictionary in formatted as constant:line_number.
  Navigate to that line number and determine if the constant holds intrinsic
  value. If it does, keep it. If it doesn't, create a #define at the top of the code and replace the constant with it's new constant. Be sure to go through all
  dictionary elements.
  },
  {"role": "user", "content": entire_code}
```

I've been using this for my homework and it's worked pretty well over the semester. While this definitely could be automated using hard code and not API's, it still is an interesting way to manipulate API requests into creating more accurate results.