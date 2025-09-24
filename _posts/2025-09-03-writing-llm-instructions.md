---
layout: post
title: "Prompts vs. Code:  The Cost of No Compiler"
comments: true
tags: [LLM, prompts]
---

 Context: People are writing no code agents where [execution instructions](https://learn.microsoft.com/en-us/microsoft-365-copilot/extensibility/declarative-agent-instructions#example-instructions) or prompts are given in English. These instructions are like pseudo code with conditionals and even loops (*repeat step 1*). This is an actively evolving area and here are a few drawback that I have observed when compared to using code.

A few days ago, I opened an agent instruction that had been in production for months. The prompt was nearly a thousand words long, stitched together through dozens of incremental edits. I counted 17 separate instructions marked *MANDATORY* scattered unpredictably across the text. The agent still worked, but reading the prompt felt like trying to make sense of a team’s sticky notes shuffled out of order.  

That experience brought out an uncomfortable truth: prompt instruction readability degrades as they evolve. They bend under the weight of iteration, and unlike code, they lack the structural guardrails that keep complexity in check.  

As I work on prompt templates that will be run thousands of times, I keep hitting the same pain points. These are issues where a small piece of code would be clearer and safer than a sprawling prompt.  

Of course, LLMs shine at NLP heavy tasks. But outside those, the differences between “prompt logic” and “code logic” become painfully obvious.  


## 1. Instruction Creep: When Anything Goes Anywhere

Prompts evolve through many rounds of tweaking. Over time, instructions accumulate in arbitrary places, without any enforced structure. The result is a prompt that works but is difficult to read or reason about.  

- **Free-form placement**: Authors can drop new conditions almost anywhere, often with just a few words of context.  
- **Ad-hoc fixes**: When a data point breaks the prompt, the typical fix is to bolt on another conditional—sometimes flagged with an emphatic *MANDATORY*. I’ve seen prompts with 17 such “MANDATORY” instructions.  
- **Inconsistent guidance**: There are several resources for software development - linters, IDEs, design patterns, etc. Such tooling is not available for prompts. There are a lot of prompting guides on the internet but there is no standard. Prompt writers often follow what worked for them in the past. Two writers may not agree on the same format. For e.g. [Azure’s best practices](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/prompt-engineering?tabs=chat#repeat-instructions-at-the-end) suggest repeating instructions at the beginning and end of a prompt to offset recency bias. But since this is soft guidance, some authors follow it and others don’t. After multiple contributors iterate, the result is a patchwork of conflicting styles.

With code, the compiler enforces discipline. It forces you into a consistent structure and throws an error if you stray. With prompts, there’s no such safeguard; mess builds silently until readability collapses.  

---

## 2. Jargon: The Silent Prompt Killer  

Unlike code, prompts don’t error out. LLMs will generate *something* even if they don’t understand your terminology. This makes company jargon invisible landmines.  

- **Blind token prediction**: LLMs are designed to produce the next token whether or not they know the context. They won’t stop and ask for clarification.  
- **Example**: I’ve seen prompts like *“If the data is coming from the cascade database, remove the PII.”* Here, *cascade* is an internal database name the model can’t possibly know. The instruction also leaves *what counts as PII* and *how to remove it* undefined. The model just guesses.  
- **Illusion of correctness**: Because no error is raised, users assume the system “understood” their intent. In reality, the result is closer to a coin toss.  

With code, a missing variable or undefined function throws an error immediately. You’re forced to be explicit—say, by using a PII removal library that requires you to specify the expected patterns. Prompts lack this feedback loop.  

---

## 3. Refactoring Without a Safety Net  

Prompt behavior is brittle, especially compared to code. Small edits can cause outsized and unpredictable shifts in results.  

- **Sensitivity to details**: Word order, synonyms, formatting choices, even an extra space after a colon—all can change outcomes.  
- **LLM-specific quirks**: Studies [1](https://arxiv.org/pdf/2310.11324),[2](https://arxiv.org/pdf/2411.10541), and [Azure guidance](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/prompt-engineering?tabs=chat#best-practices) show these quirks vary across models. What works for one LLM often breaks on another.  
- **Refactoring risk**: Even with strong test coverage, restructuring a prompt is daunting. You may never recover the same eval scores once wording shifts. The messy prompt often “wins” by default.  

By contrast, code with good test coverage can be refactored confidently. Tests enforce equivalence. In prompt engineering, no such safety net exists.  

---

## Closing Thought  

LLMs remove the hard edges of programming: no compiler errors, no enforced structure, no type checks. That flexibility is what makes them powerful—but also what makes prompts messy, fragile, and hard to maintain at scale. When clarity and reliability matter, code is still the safer language.  