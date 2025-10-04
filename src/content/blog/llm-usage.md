---
date: "2025-10-03"
title: "How I use Simon Willison's LLM"
pubDate: 3 October 2025
description: Large Language Model usage
slug: llm-usage
---

As those who've had to listen to me for any length of time will know, I'm a big fan of Simon Willison's [llm](https://github.com/simonw/llm) package. To avoid cluttering up my blog posts with too much detail, I'm jotting down notes on how I set up and use the package here.


### Prompting and reading

One big disadvantage of a terminal-based LLM tool is that typing in the terminal sucks! And so does reading terminal output, for that matter. The answer to this is to offload as much of the writing and reading to text files as possible. Last year, I built a project that was a fun but probably over-engineered solution -- [a package](https://github.com/matweldon/markdown_llm) that enables chatting with LLMs in a markdown file. Unfortunately the package broke after an upgrade to `simonw/llm` and I haven't had the chance to fix it yet.

However, a simpler solution is just to rely on piping to and from the terminal. I usually create a file called 'prompt.md', into which I write my prompt, and then I direct the text into the tool using `< prompt.md`. If the response is long and contains code, I can direct it into an output file using `> output.md`. I can combine them so that the whole command looks like 

```bash
llm -m gpt-4.1 < prompt.md > output.md`
```

The idea here isn't to save a copy of the conversation for posterity -- there's no need for that because `llm` logs every conversation to a searchable SQLite database. So I usually write over both 'prompt.md' and 'output.md' and have them in my '.gitignore'. It's just about being able to easily write prompts and read responses.

The SQLite database provides another option for reading responses: rather than piping the output directly to a file you can pipe the result of 

```bash
llm logs -n 1 > output.md
```

which is useful if you forget to pipe the output when you first run the command, and contains a bit more metadata in the header. It's also a good way to share a whole conversation with others by doing 

```bash
llm logs -c > conversation.md`
```

To pipe code context into the terminal, Simon's package has several more advanced features such as `files-to-prompt` and fragments, but I usually just use simple commands. For example, to isolate lines 50 to 75 of a code file it's easy to do: 

```bash
sed -n 50,75p file.py | llm -s "Describe this function"
```

Or, to incorporate the code alongside a prompt, pipe it into the prompt first 

```bash
sed -n 50,75p file.py >> prompt.md 
llm < prompt.md > output.md
```

### Setting up a GitHub Codespace

I have a 'dotfiles' GitHub repo, with an 'install.sh' including the lines:

```bash
pip install uv && uv tool install llm 

llm install llm-github-models 
llm install llm-gemini
llm install llm-anthropic
llm models default github/gpt-4.1
```

This installs `simonw/llm` into every GitHub Codespace I create, and allows me to use GitHub models (which are OpenAI models) for free. The other option, as recommended by Simon, is to add these lines to the `postCreateCommand` field in `devcontainer.json`.

### Why do I like it so much?

This is a really good question! After all, GitHub Copilot, Claude Code, Codex and others are all available in at least some of my coding environments, and do a lot more of the heavy lifting for me. I think the answer is that `simonw/llm` is available _in all of my coding environments_. Setting it up is as simple as installing a python package (which is easy with uv) and finding some API key. If I can't access my favourite Anthropic models, I can use OpenAI models, or Gemini models, or even local models.

And the extra effort to occasionally copy and paste code feels worth it for the chance to closely review what the model is adding to my code. Having to use terminal commands keeps me closer to controlling the context and the workflow. This kind of workflow is also much, _much_ cheaper in terms of tokens compared to using an agentic tool.