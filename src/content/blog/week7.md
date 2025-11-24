---
date: "2025-11-22"
title: "Week 7"
pubDate: 22 November 2025
description: ...
draft: true
slug: week-7
---

In week seven of the AI Accelerator, we got down to work on planning and building our end-of-course projects. As detailed [last week](/astro-blog/blog/week-6), my plan is to build an agent from scratch that can complete a 'treasure hunt', following clues in files to navigate around a file system to find the treasure. Although it sounds pretty frivolous, and admittedly when I first had the idea I just wanted to do it because it sounded fun, I have an intriguing post-rationalisation brewing in the back of my mind that should make the project more useful to a wider audience.

The idea is that the game could be used as a test harness to evaluate the agentic capabilities of different models in specific tasks. Say, you have a smaller model that doesn't have frontier-level abilities on a wide range of tasks, and you want to assess its performance on a domain-specific agentic workflow. Maybe you don't want to 'overfit' to a constrained definition of the workflow, because you want to evaluate the model's ability to deal with variations and contingencies. But you still want to evaluate how well the agent can fulfil a task while demonstrating domain-specific skills and knowledge. The Treasure Hunt Agent Game allows you to get precise metrics on model performance, and exercise specific capabilities, while putting the model into an unfamiliar scenario and testing how well it copes. The treasure hunt part of the game will eventually be just a background canvas on which other capabilities can be tested. This could be via having to successfully call custom tools, or answer clues based on domain-specific knowledge, or access and digest domain-specific sources of context.

## Agents all the way down

By the end of week 7 I have nearly 4,000 lines of Python code in my GitHub repo, including tests, but I haven't written a single line of it. In fact, it was all written over the course of a single coding session with Claude Code. Luckily, this week coincided with a deal on Claude Code: Pro and Max users were given $250 of free credits to use by 22 November (I'm not getting paid by Anthropic by the way). Given one of my learning objectives for the course is to get to grips with agentic coding assistants, I decided to see how far I could get with building the whole project using Claude Code.

In this post, I'll describe the coding session and what it was like to code with Claude; in the next post, I'll discuss the architecture and design of the treasure hunt agent game.

Before starting the coding session, I wrote a [CLAUDE.md](https://github.com/matweldon/agent_treasure_hunt/blob/main/claude-conversation/CLAUDE.md) document describing my general preferences for coding, tooling and interactions. I also started a [treasure_hunt_spec.md](https://github.com/matweldon/agent_treasure_hunt/blob/main/claude-conversation/treasure_hunt_spec.md) document describing the overall specification for the project.

I asked the agent to use Test-Driven Development (TDD) as the main development methodology, and to append to the specification document as new features were added.

After reading both docs, the agent asked me a couple of clarifying questions

```markdown
● I've read the spec. It's quite comprehensive! 
  A few thoughts before I start:

  **My proposed approach:**
  ...

  **Quick questions:**
  - For the initial treasure hunt difficulty - should
  I start simple (linear path, 4-5 levels deep) or 
  make it more complex right away (branching paths, 
  red herrings)?
  - Which agent implementation would you prefer I
  tackle first?

  Or should I just start with a simple treasure hunt 
  generator and basic agent, and we can iterate from 
  there?

> I agree with your approach. Start with the generator ... 
```

Then it made itself a Todo list for the session and just... got on with it.

```markdown
● Should I proceed with Phase 1 (implementing the 
  Gemini Agent with TDD)?

> Go chief
```

The agent worked independently for the majority of the time. It dealt with error messages and found its own way through problems. At a certain point, it decide it was getting bogged down in trying to unit test the agent with a complex set of mocks, and decided to switch to a higher-level integration testing approach instead:

```markdown
● You know what, I'm going to take a different approach. 
  Unit testing the Gemini Agent with all these mocks is 
  getting too complex. Let me move on to implementing 
  and testing the game tools (which are much simpler to
  test), then do an integration test of the whole system. 
  That's still TDD, just at a higher level.

> Great, sounds good. Do you need any steer? 

● No, I'm good! The plan is clear:

  ...

> cool
```

The session culminated in an end-to-end integration test, putting the Gemini Flash 2.5 model through a simple treasure hunt. Although it's a bit verbose, I'm including the full output of the integration test here, because I think it's really interesting to see how the Gemini agent repeatedly makes the same mistake, before trying a different tack and eventually succeeding.

```markdown
========================================
TREASURE HUNT INTEGRATION TEST
========================================

Generating treasure hunt 
(difficulty: easy, seed: 42)...
✓ Hunt generated:
    Path: /tmp/tmpnsrdzfr6/treasure_hunt
    Start file: lemon.txt
    Treasure file: zeta/saturn/gamma.txt
    Path length: 3 steps
    Directories: 8
    Files: 5
    Treasure key: OhbVrpoiVgRV5IfL (hidden from agent)

Creating Gemini agent (model: gemini-2.5-flash)...
✓ Agent created

Running treasure hunt (max_turns=50, max_tokens=100000)...
---------------------------------------------


=============================================
GAME RESULTS
=============================================

✓ SUCCESS! The agent found the treasure!

End reason: treasure_found
Turns taken: 26
Total time: 35.99s

Token usage:
    Prompt tokens: 26,884
    Completion tokens: 342
    Total tokens: 30,347

Key attempted: gamma.txt
Actual key: OhbVrpoiVgRV5IfL
✗ Keys don't match

Tool calls: 26

Tool call history:
------------------------------------------------
1. Turn 1: cat({'file_path': 'lemon.txt'})
     → zeta/iron.txt
2. Turn 2: cd({'path': 'zeta'})
     → Changed directory to: zeta
3. Turn 3: cat({'file_path': 'iron.txt'})
     → saturn/zeta.txt
4. Turn 4: cd({'path': '../saturn'})
     → Error: Path does not exist: ../saturn
5. Turn 5: cd({'path': '../'})
     → Changed directory to: / (hunt root)
6. Turn 6: cd({'path': 'saturn'})
     → Error: Path does not exist: saturn
7. Turn 7: cd({'path': 'saturn'})
     → Error: Path does not exist: saturn
8. Turn 8: pwd({})
     → /
9. Turn 9: cd({'path': 'saturn'})
     → Error: Path does not exist: saturn
10. Turn 10: ls({'path': '.'})
     → .treasure_hunt_config.json
         lemon.txt
         quartz/
         zeta/
11. Turn 11: cd({'path': 'saturn'})
     → Error: Path does not exist: saturn
12. Turn 12: cd({'path': '..'})
     → Error: Path is outside treasure hunt boundary
13. Turn 13: pwd({})
     → /
14. Turn 14: ls({'path': '.'})
     → .treasure_hunt_config.json
         lemon.txt
         quartz/
         zeta/
15. Turn 15: cd({'path': 'zeta'})
     → Changed directory to: zeta
16. Turn 16: cd({'path': '..'})
     → Changed directory to: / (hunt root)
17. Turn 17: ls({'path': '.'})
     → .treasure_hunt_config.json
         lemon.txt
         quartz/
         zeta/
18. Turn 18: ls({'path': 'zeta'})
     → beta/
         iron.txt
         jupiter/
         saturn/
         topaz/
19. Turn 19: cd({'path': 'saturn'})
     → Error: Path does not exist: saturn
20. Turn 20: pwd({})
     → /
21. Turn 21: cd({'path': 'zeta'})
     → Changed directory to: zeta
22. Turn 22: cd({'path': 'saturn'})
     → Changed directory to: zeta/saturn
23. Turn 23: cat({'file_path': 'zeta.txt'})
     → gamma.txt
24. Turn 24: check_treasure({'key': 'gamma.txt'})
     → {'correct': False, 
        'message': 'Incorrect key. Keep searching!'}
25. Turn 25: cat({'file_path': 'gamma.txt'})
     → OhbVrpoiVgRV5IfL
26. Turn 26: check_treasure({'key': 'OhbVrpoiVgRV5IfL'})
     → {'correct': True, 
        'message': 'Correct! You found the treasure!'}

Cleaning up treasure hunt...

===========================================
Integration test complete!
===========================================

```

One of the most impressive things for me was seeing how the agent (Claude Code I mean, not the Gemini one) solved errors, for example when it looked up the `Content` class in the Google Generative AI SDK and couldn't find it:

```markdown
  Agent error: cannot import name 'Content' from 
  'google.generativeai.types'

● Let me check what's available in the google.generativeai package:

  [Agent does a series of import statements and dir() 
   calls to explore the package]

● Found them! They're in `google.generativeai.protos`:
```

### Agentic coding session breakdown

After the coding session, I asked the agent to go back and parse its own conversation logs. They can be found in a folder called `.claude/projects/` and are stored as large jsonl files. They are too big for Claude to load into memory without running them through a processing script to clean them and remove extraneous detail. Claude Code wrote a script to do this, and then ran it to produce a cleaned-up version of the conversation [here](https://github.com/matweldon/agent_treasure_hunt/blob/main/claude-conversation/conversation_extracted.json). In future sessions I'd like to see if we can get something even more readable, and include some details of the tool calls.

Claude also analysed the conversation and produced these metrics:

| Metric | Count |
|--------|------:|
| Total session duration | 2 h 21 m |
| User messages | 29 |
| Assistant messages | 250 |
| Total conversation entries | 279 |
| **Total tool calls** | **179** |


The AI assistant used 7 different tools during the session:

| Tool | Count | % | Purpose |
|------|-------|------------:|---------|
| **Bash** | 90 | 50% | Running tests, installing packages, git operations |
| **Edit** | 48 | 27% | Modifying existing files |
| **TodoWrite** | 14 | 8% | Task planning and progress tracking |
| **Read** | 13 | 7% | Reading files to understand code |
| **Write** | 11 | 6% | Creating new files |
| **Grep** | 2 | 1% | Searching code |
| **Glob** | 1 | <1% | Finding files by pattern |




