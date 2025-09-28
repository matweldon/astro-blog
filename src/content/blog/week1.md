---
date: "2025-09-27"
title: "AI Accelerator Week 1"
pubDate: 27 September 2025
description: Learning objectives and set-up
slug: week-1
---

The Department of Science, Innovation and Technology has launched a new training programme called the AI Accelerator, to upskill analysts and data scientists across government in the skills to work with these new technologies. I've been lucky enough to blag a place on the course. One of my self-imposed goals for the course is to improve my regular writing practice by 'working in the open' and blogging my progress each week, so here's my recap of week one.

The rationale for the Accelerator is that there is a large and growing number of analysts in government who have coding experience and write a lot of code in limited contexts, but who are now finding that the new technologies require a great deal more software engineering skills to handle cloud computing, APIs and the like, and need some help to get to the next level. As I've learned during my time in the Data Science Campus, there's a chasm between the coding skills needed to write a notebook to do some statistics and make some charts, and the skills needed to build and maintain an AI app in production. 

I'm very fortunate to have been given the opportunity to participate in the programme. I feel like my teammates and I have had to jump in at the deep end and teach ourselves to swim with AI engineering, so two days per week to focus on filling in the gaps and consolidating what I think I've learned is a massive luxury. We've also been promised access to powerful cloud workstations with GPUs and model API keys. The course has a broad syllabus: starting with a review of statistical learning theory and machine learning basics; moving on to deep learning; LLMs and agents; essential software engineering skills; then cloud and MLOps. It sounds like a lot to cover in eleven weeks.

Before starting the course, I wrote myself some learning objectives:

#### 1. Experiment with tool-using agents, and find out how they work under the hood

I have some experience with AI-assisted coding: we have GitHub Copilot, and I've also been using Simon Willison's excellent [llm](#) tool to interact with various models in the terminal. But I've heard so many good (and bad) things about agentic AI coding assistants such as Claude Code, OpenAI Codex and others. If I achieve nothing else, I have to find out whether they live up to the hype. I'm also planning to examine the open source codebases of some of these agents to understand how the main agent loop works.

#### 2. Play with local models, including ripping their guts out

Some AI engineers are a bit too enthusiastic about local models and think they'd be easy to roll out in production settings. I'm skeptical because I don't believe that local models are competitive on total cloud costs or quality with hosted models. But I'm still curious about what I can do with them, and I'm not likely to get a better chance to play with them than now.

This includes not only fine-tuning local models, but also doing as much as possible to pull them apart and examine their structure. I'm not even sure how much is possible. There are a couple of things I'd be especially interested to investigate if possible. First, I'd like to access the logits of the next word prediction engine, to experiment with uncertainty quantification methods such as [conformal prediction](#). Second, I'd like to experiment with extracting embeddings from the representation layer of LLMs and fine-tuning instruction-tuned models for information-retrieval uses.

#### 3. Build agents to improve the data-led policy-making pipeline

This one is basically the culmination of the others, and is harder to explain. I watched [an interview](#) with members of the Claude Code team, who said something intriguing about their intention to branch off into 'general agents' -- i.e. not coding agents. AI-assisted coding has really boomed over the last couple of years, to a much greater extent than AI progress in any other professional area. The obvious reason is that AI is built by computer scientists, so they're bound to make their own tools first. 

But I don't think that's the whole picture. I think there's something about the existing culture and social working practices of software development that makes the insertion of AI agents especially easy and productive, compared to other professional spheres. I'll elaborate on that a bit in another post. For now, I'll just say that I want to see if I can build agents that help civil service policy-makers and analysts with aspects of what we call the [ROAMEF cycle](#).

### Week One
___



In week one, we got ourselves set up in our development environments, reviewed Python data manipulation and machine learning basics, and explored some resources such as Kaggle and HuggingFace. To be honest I wasn't very engaged in the Python content, but I really enjoyed setting up my VS Code environment and customising the IDE and terminal. In fact, I ended up setting up two environments because a technical glitch meant that I had to use GitHub Codespaces on one of the days. 

But I didn't mind -- it was fun 🤓! As a project lead on my past couple of projects, I felt I haven't had a chance to do enough of the everyday coding, and my muscle memory for the tools has atrophied. My role is like a cross between a lead dev and a product manager. That means I spend a lot of my time each day in meetings with stakeholders and team members, responding to messages and emails, reviewing PRs and keeping tickets on the project board moving to the right. So in short, I'm looking forward to spending more time in the terminal.

I'm also involved in conversations around dev tooling for data scientists in ONS, so I have a professional interest in seeing how different web-based development environments are set up and configured.