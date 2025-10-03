---
date: "2025-10-03"
title: "AI Accelerator Week 2"
pubDate: 3 October 2025
description: Machine learning
slug: week-2
---


The topic of the AI Accelerator this week was probability, statistical learning and machine learning algorithms. Although this was a review for me of material I learned, I found that I learned more than I expected. The approach to learning was innovative, using a flipped learning style augmented by hands-on work with AIs.

After a brief overview of the topic, the course leader gave us a task to complete with the help of LLMs, and then that was it -- we were left to our own devices. The task was to implement a machine learning algorithm _from scratch_, which could be either a decision tree, linear regression or K means clustering. I chose a decision tree classifier because it was the one I was least familiar with the internal workings of. We were given access to the OpenAI API key.

Now, at this point you might be thinking: if you ask any capable model to create a decision tree model it will just spit out some code that works. So how does it help me learn? The key is what I'd call the 'prompt, review and tinker' method.

After installing `simonw/llm`. 

> I'd like you to write a decision-tree classifier for me with a train method and a predict method. It's for learning so it should be very simple with no features apart from the guts of the algorithm wrapped in a simple class. The classifier train method should take a matrix X and a binary array y. The predict method should just take a matrix X. Just write python code with no explanation or even fenced code blocks. I'm going to pipe the result directly into a file so it's important you DON'T ADD ANY TEXT OR WRAPPER.

* My task was to implement a decision tree classifier from scratch
* Implemented, tested with some fake data - added notes to understand it
    * Recursive functions - sweet!
* Thought, why not create a random forest
* Leaf size
* Probabilities as well as modal prediction
* Look for a dataset on HuggingFace
* Found one, how do I download it?
* My classifier couldn't handle continuous values, need to bin in quantiles
* But wait, how is it productionisable?
* The quantization needs to be baked into the model!
* Using Claude Code - it's impressive, but keep getting 429 errors
* I used Claude Code to add serialisation methods to the models

### Reflections on using AI to learn

The copy-and-paste pair programming mode feels better for learning than the agentic mode. Simon W's package is especially good because it's almost agentic (you can pipe content to and from files) but requires careful thought.

It's true that there will be some things that I don't learn -- there's no such thing as a free lunch. But those usually correspond to the things I don't want to learn. For example, I've never learned how to properly use matplotlib, and now I don't think I ever will because I don't need to. Fiddling with aesthetic settings is not the kind of data scientist I am, but I know others would differ.