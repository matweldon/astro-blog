---
date: "2025-10-03"
title: "AI Accelerator Week 2"
pubDate: 3 October 2025
description: Machine learning
slug: week-2
---


This week's AI Accelerator covered probability, statistical learning and machine learning algorithms. Most of this was material I'd encountered before, but the way we approached it meant I found that I learned more than I expected. The course employed a flipped learning model, combining brief theoretical overviews with extensive hands-on implementation work supported by large language models.

### The assignment

After a brief overview of the topic, the course leader gave us a task to complete with the help of LLMs, and then we were left to our own devices. The task was to implement a machine learning algorithm _from scratch_, which could be either a decision tree, linear regression or K means clustering. We were given access to the OpenAI API key.

I chose to build a decision tree classifier, primarily because I had the least familiarity with its internal workings. It turned out to be a good choice, because it was both extensible and computationally demanding, a combination that forced me to reflect on the subtleties of computation.

Now, at this point you might be thinking: if you ask any capable model to create a decision tree model it will just spit out some code that works. So how does it help me learn? The key is what I'd call the 'prompt, review and tinker' method: get something that works; read it carefully, annotate it; and based on that understanding iterate with a combination of manual and AI-assisted improvements.

I began by installing Simon Willison's `llm` [command-line tool](https://matweldon.github.io/astro-blog/blog/llm-usage/), which provides a streamlined interface to various language models. Then I typed this into my 'prompt.md':

> I'd like you to write a decision-tree classifier for me with a train method and a predict method. It's for learning so it should be very simple with no features apart from the guts of the algorithm wrapped in a simple class. The classifier train method should take a matrix X and a binary array y. The predict method should just take a matrix X. Just write python code with no explanation or even fenced code blocks. I'm going to pipe the result directly into a file so it's important you DON'T ADD ANY TEXT OR WRAPPER.

and ran `llm -m gpt-4.1 < prompt.md > dtree.py`. Piping output directly into a python file is my equivalent of YOLO mode. It's funny because I'm fairly sure it takes me longer to admonish the model not to add backticks, than it would for me to just copy and paste the code. But it gives big-dev energy so I like it.

What I got back was pretty clean -- a basic implementation that used information gain to figure out where to split the tree, built everything recursively, and then had a predict method that walked through the tree structure.

## Prompt, review and tinker

First, I went through and added comments to explain what each part was doing. This sounds simple but it's actually where a lot of the learning happened, because you can't write a good comment about something you don't understand. I had to work through how information gain actually determines the best split, and how the recursive function builds up the tree structure. The recursion part especially clicked for me in a way it never had before.

I asked the model to simulate some fake data on two dimensions, because I wanted something that I could easily plot. It looked like it was doing roughly what it should:

[picture](#)

Once I felt like I had a grip on the basic implementation, I started extending it. First thing I tried was building a random forest, because what's the use of a single decision tree? I added some other features too:

* I decided the random forest especially would benefit from being able to set a minimum leaf size, to regularise predictions

[pictures](#)

* I wanted to see in more detail how the model made its predictions, so I added the ability to make probability predictions instead of just a single modal prediction

[pictures](#)

Each of these extensions was easy to add with the help of LLMs but forced me to really understand what was going on in the original implementation.

### Testing the model on real data

Although it was fun to draw pictures with fake data and a known ground truth, at some point I wanted to check that it would work reasonably on real data. Something I haven't mentioned until now is that the course leader heavily hinted that in a couple of weeks we'll be deploying these home-made models as APIs. That means the model has to work with data 'in the wild' and needs to be deployable after training.

I browsed the HuggingFace datasets directory and eventually found a medium-sized [dataset](#) of house attributes and prices. I decided that the data was a good candidate for a prediction use case. I could convert the house price to a binary measure, and then train a model to predict whether a house costs more than $1 million based on attributes such as bedrooms, floor area and local amenities.

At first, I downloaded the dataset and started working on it in the same way I would if I was writing an analytical pipeline: build a set of scripts to load, clean and manipulate the data before training the model on it and then validating the fit. An important pre-processing step was to bin many of the features of the house prices dataset into quantiles. This was necessary because my inefficient implementation of decision trees loops through every possible candidate threshold of every feature, which would quickly become intractable with many continuous variables.

But then it struck me that the analytical workflow I was used to was completely inappropriate for an MLOps pipeline. If someone wants to use my model and get a prediction, they don't want to have to clean, manipulate and quantize the data before sending it to my API -- they want my API to meet the data where it is, and do the work to make a valid prediction.

So I had to refactor it to bake the preprocessing into the model itself. The quantile thresholds set during training needed to be saved as part of the model so they could be applied consistently to new data.

The final task to make the model usable in an API was to make the trained model serialisable -- if it's to be run in a Flask or FastAPI server it'll need to be loaded from disk or cloud storage quickly and reliably. At this point I finally had Anthropic access through the Google Vertex API, so I switched to using Claude Code. Unfortunately the API was unreliable and frequently threw 429 errors due to the volume of requests, which forced Claude Code to use retries to complete tasks. However, in the end it managed to add proper serialization so the model could save and load everything it needed -- both the tree structure and all the preprocessing parameters.


### Reflections on using AI to learn

The tool you use matters more than I expected. Simon Willison's `llm` package sits in this interesting middle space where it's automated enough to be convenient -- you can pipe content to and from files -- but it's not so automated that you can zone out. Every step requires you to do something deliberately, which naturally makes you think about what you're doing.

[picture of llm](#)

In contrast, when I used Claude Code with Sonnet 4.5 through Google cloud platform, it felt as if I almost had nothing to do, except wait for the retries as the Google Vertex API frequently failed to cope with the throughput. Cost is another factor that distinguishes non-agentic from agentic uses of AI for coding. I used Claude Code for only about 45 minutes, and it cost nearly £3 in tokens.

[picture of claude code](#)

It's true that using LLMs will have hidden some subtlety from me that I won't even realise I've missed. But I feel that using the LLM allows me to focus on the parts of the process I'm curious about, and ignore details I'm less interested in. For instance, I've never properly learned any plotting libraries and now I'm pretty sure I never will because with LLMs I don't need to. Other people might feel differently about that and that's fine -- everyone's got different things they actually need to know versus things they can afford to treat as black boxes.

What I did learn felt substantial though. Not just about decision trees and random forests, but also about the capabilities and limitations of different large language model tools for development and learning.