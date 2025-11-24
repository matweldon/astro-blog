---
date: "2025-11-15"
title: "Week 7"
pubDate: 15 November 2025
description: ...
draft: true
slug: week-7
---

In week 7 of the AI Accelerator, we got down to work on planning and building our end-of-course projects. As detailed [last week](/astro-blog/blog/week-6), my plan is to build an agent from scratch that can complete a 'treasure hunt', following clues in files to navigate around a file system to find the treasure. Although it sounds pretty frivolous, and admittedly when I first had the idea I just wanted to do it because it sounded fun, I have an intriguing post-rationalisation brewing in the back of my mind that should make the project more useful to a wider audience.

The idea is that the game could be used as a test harness to evaluate the agentic capabilities of different models in specific tasks. Say, you have a smaller model that doesn't have frontier-level abilities on a wide range of tasks, and you want to compare its performance to that of more capable models on a domain-specific agentic workflow. Maybe you don't want to 'overfit' to a constrained definition of the workflow, because you want to evaluate the model's ability to deal with variations and contingencies. But you still want to evaluate how well the agent can fulfil a task while demonstrating domain-specific skills and knowledge. The Treasure Hunt Agent Game allows you to get precise metrics on model performance, and exercise domain-specific skills and knowledge, while putting the model into an unfamiliar scenario and testing how well it copes. The treasure hunt part of the game will eventually be just a background canvas on which other capabilities can be tested. This could be via having to successfully call custom tools, or answer clues based on domain-specific knowledge, or access and digest domain-specific sources of context.

## Agents all the way down

By the end of week 7 I had about 1,600 lines of Python code in my GitHub repo, including tests, but I hadn't written a single line of it. In fact, it was all written over the course of a single three-hour coding session with Claude Code


```markdown
● You know what, I'm going to take a different approach. Unit testing the Gemini Agent with all these mocks is getting too complex. Let me move on to implementing and testing
  the game tools (which are much simpler to test), then do an integration test of the whole system. That's still TDD, just at a higher level.

● The GeminiAgent implementation is done. Let's move to the game tools which are much simpler to test - they're just file operations with path validation. I'll write those tests
   next.

> Great, sounds good. Do you need any steer? 

● No, I'm good! The plan is clear:

  1. Write tests for the game tool functions (ls, cd, cat, pwd, check_treasure, give_up, ask_human)
  2. Implement them with proper path validation against treasure_hunt_root
  3. Write tests for the TreasureHuntGame loop
  4. Implement the game loop
  5. Do an end-to-end integration test with a real treasure hunt + Gemini API

  The tools are much simpler to test than the agent - just regular file operations with boundary checks. I'll get started on those now.

> cool
```