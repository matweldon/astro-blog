---
date: "2025-11-08"
title: "Week 6"
pubDate: 8 November 2025
description: ...
draft: false
slug: week-6
---

After a break for half term we are back into the AI Accelerator programme. Those of you who have been following my blogs will notice that I've skipped straight from week 3 to week 6. That's because I'm a terrible blogger and I couldn't get my act together to publish anything. Apologies for that, and if I ever make a commitment to publish a weekly blog again, please remind me to get my head examined! I did actually write a rough draft for week 5 and I'm hoping to polish it up and publish it before next week.

This week we were introduced to the concept of AI agents, and we had a go at building our own using LangChain and a range of models. Simon Willison [defines agents](https://simonwillison.net/2025/Sep/18/agents/) as: "An LLM agent runs tools in a loop to achieve a goal." This highlights two features that distinguish agents from simpler LLM 'workflows': firstly, agents have access to tools which they can use to gather information and manipulate their environment; secondly, agents operate in a loop to achieve a goal, meaning that the loop continues until the goal is achieved. And the loop can continue for much longer than a single prompt/response interaction.

An important corollary of tool use is that agents gather information and in effect construct their own context as they go along. This is in contrast to simpler LLM workflows where the context is limited to the prompt and any additional context provided by the user. The task of prompt engineering, which has up to now meant finely crafting the LLM's entire information environment, is now replaced by the task of designing tools so that the agent can gather the information it needs, when it needs it.

The Anthropic website has some great resources giving advice on [building effective agents](https://www.anthropic.com/engineering/building-effective-agents) including notes on using their [agent development kit (ADK)](https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk). They underline the same point about context:

> The file system represents information that _could_ be pulled into the model's context. When Claude encounters large files, like logs or user-uploaded files, it will decide which way to load these into its context by using bash scripts like grep and tail. In essence, the folder and file structure of an agent becomes a form of context engineering.

This week I also had another chance to experiment with the Claude Code command line interface. Using the tool this time, I really noticed how well the model could gather context by exploring and navigating the file system. It inspired me to think about building an agent that could explore a virtual environment, gathering context as it went along, and using that context to make decisions about where to go next. This is the basis of my end-of-course project, which I'll introduce in the next section.

## Introducing my end-of-course project: Treasure Hunt Agent

Actually, the origins of this idea go back to the pandemic, when I was teaching my kids a bit of programming at home. I wanted them to learn about how to navigate the file system, and I thought about creating a simple treasure hunt game where they would have to find clues hidden in different directories and files. The idea was to create a virtual environment that they could explore using command line commands such as `cd`, `ls`, and `cat`. Each clue would lead them to the next location, until they finally found the treasure. I never got around to building it, but the idea came back to me when I read the Anthropic article about agents exploring file systems.

My plan is to build the treasure hunt game as an evaluation tool for the agent capabilities of different models. The game will consist of:

- a random game generator that creates a complex, deeply-nested virtual environment with directories and files containing clues
- a limited set of tools that the agent can use to explore the environment (e.g. `cd`, `ls`, `cat`, `grep`, etc.)
- an interface class that allows a user to instantiate an agent with a given model and set of tools, and run the agent in the game environment
- a set of evaluation and monitoring utilities

I'll start experimenting with capable frontier models such as Gemini, Claude and GPT-5, but one of the goals of the project is to explore how well smaller open source models can perform as agents in this environment, possibly after some fine-tuning.

The game will be designed to be extensible so that new clue types, tools and challenges can be added. For example, the user might add special tools that simulate business-relevant capabilities such as querying a database or calling an API. Maybe the agent will have to call these tools in the correct way or sequence in order to get the next clue.

Maybe the game will just be a bit of fun, and at the very least it will help me learn more about building agents and working with different models. But it might also turn out to be a useful way to evaluate the capabilities of different models in a controlled environment. I'll keep you posted on my progress!

