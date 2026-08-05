---
layout: post
title:  "Instructions vs. Skills vs. Agents: Knowing the Difference Will Save You Tokens"
date:   2026-07-29 12:00:00 +1000
categories: [ai, tooling]
tags: [claude, github-copilot, ai-agents, prompt-engineering, developer-tools, context-window]
permalink: /blog/2026/Instructions-Skills-and-Agents/
---

I keep running into the same setup when I look at how people configure their coding assistants: one giant instructions file with everything crammed into it. Coding standards, one-off workflows, edge-case rules that only apply once a month - all of it loaded into context on every single request, whether it's needed or not.

It's a habit, not a strategy, and it's an expensive one. Instructions, skills, and agents solve three different problems. Mix them up and you either bloat your context window with stuff you didn't need this turn, or you assume something will "just work" that was never going to trigger in the first place.

<!--more-->

Here's the actual distinction, stripped of the marketing language: it comes down to *when* content enters the model's context and *how long* it sticks around.

- **Instructions** load every turn, relevant or not.
- **Skills** sit off to the side as a name and a one-line description until the model decides they're worth pulling in.
- **Agents** hand the work off entirely - a separate context does the digging, and you only see what comes back.

Get the boundaries wrong and you'll notice it two ways: either your system prompt is so bloated there's no room left for the actual problem, or your assistant has no idea about a convention you were sure it would pick up automatically.

---

## 1. Instructions

Instructions are static and always-on. They go into every request no matter what you're asking - `CLAUDE.md`, `.github/copilot-instructions.md`, a system prompt, doesn't matter which.

```markdown
# CLAUDE.md
- Use tabs, not spaces
- Never commit directly to main
- All API responses must use the ApiResult<T> wrapper
```

**The Pro:** It's guaranteed to apply. There's no "the model decided this wasn't relevant" - if it's in instructions, it's in the prompt, every time.

**The Con:** Every token in that file gets paid for on every turn, including the vast majority of turns where it does nothing. A 3,000-token instructions file isn't a one-off cost - it's 3,000 tokens gone from every single exchange in a long session, and that adds up fast against real work like code and history.

**Best For:** Rules that are genuinely universal - coding standards, hard safety constraints ("never force-push"), the stuff you'd want applied to 100% of tasks in the repo. If something only matters one time in ten, it doesn't belong here.

---

## 2. Skills

Skills invert the model. Instead of always loading, the model sees a short list of names and descriptions up front, and only pulls the full content in when it looks relevant to what's being asked. Claude's `Skill` tool and Copilot's reusable prompt files are both built on this idea, even if the plumbing underneath is different.

```markdown
---
name: deploy-checklist
description: Use when deploying to production - runs the pre-deploy verification steps
---

1. Run the full test suite
2. Check the changelog is updated
3. Confirm the feature flag is behind a gate
...
```

**The Pro:** Practically free when you're not using it. You pay for a one-line description until the moment a skill actually fires, at which point the full content loads for that turn only. That means you can build up dozens of specialised workflows without any of them taxing your always-on budget.

**The Con:** Nothing guarantees it fires. The model has to correctly guess relevance from a short description, and a vague or lazy description means a genuinely useful skill just never comes up - or the wrong one does. The name and description carry far more weight here than they do for instructions.

**Best For:** Things you reach for often but not constantly - a deployment checklist, a specific review rubric, a migration procedure specific to one repo. Anything conditional, not universal.

---

## 3. Agents

Agents are delegation, not a way of keeping content out of context entirely. An agent can be pre-loaded with its own instructions - a frontend agent might already know which folder the app lives in, its component conventions, its build tooling - the same way `CLAUDE.md` primes your main session. The difference is *where* that context lives: it sits in the agent's own window, not yours, and all you get back is a summary. So the real question isn't whether context gets loaded, it's whose window it loads into.

```
"Research how the auth middleware handles token refresh
across the codebase, then report back in under 200 words."
→ a fresh agent does the digging in its own context - the file
  reads, the grep results, the dead ends - and you only see
  the summary at the end
```

**The Pro:** It keeps the mess out of your main thread. Dozens of file reads and dead-end searches stay contained in the sub-agent's context; you only pay for the distilled answer. That's what makes a long, complicated session survivable without hitting a wall.

**The Con:** Whatever an agent knows about the codebase up front, it still starts cold on *your conversation* - it has no idea what you've already tried or ruled out unless you brief it, so a lazy prompt gets you a shallow, generic result. There's also a real cost and latency hit - spinning one up to answer something you could look up in ten seconds is usually the slower, more expensive option.

**Best For:** Open-ended digging that would otherwise flood your context with noise - broad codebase searches, multi-file investigations - and independent sub-tasks you can run in parallel. Not for anything you need an answer to before your very next move.

---

## The Comparison at a Glance

| Feature | Instructions | Skills | Agents |
|---|---|---|---|
| **When loaded** | Every request, always | On demand, when judged relevant | Never loaded into your context at all |
| **Context cost when unused** | Full cost, every turn | Basically nothing (one-line description) | None |
| **Context cost when used** | Same as always | Full skill content, that turn only | Just the summary that comes back |
| **Reliability** | Guaranteed | Depends on the model guessing relevance correctly | Depends on how well you briefed it |
| **Best for** | Rules that apply to everything | Workflows you use sometimes, not always | Exploration and parallel sub-tasks |
| **Fails how** | Bloated context, crowds out real work | Silently never triggers | Slow and costly for trivial lookups; shallow if the prompt is thin |

---

## Claude vs. GitHub Copilot: Where They Actually Differ

The three-layer split holds for both tools, but the details underneath aren't the same.

**Instructions** look almost identical either way. Claude Code reads `CLAUDE.md` at both a global (`~/.claude/CLAUDE.md`) and project level, both loaded automatically every time. Copilot spreads the same idea across `.github/copilot-instructions.md` at the repo level plus personal custom instructions in VS Code/GitHub settings. Different filenames, same "always injected" behaviour.

**Skills is where they part ways.** Claude treats skills as their own first-class thing, separate from instructions - discoverable by name and description, scoped globally or per-project, picked up automatically when relevant. Copilot doesn't really have a matching feature. The nearest things are reusable **prompt files** (`.github/prompts/*.prompt.md`) and **custom chat modes**, but you mostly invoke those yourself, slash-command style, rather than the model discovering and loading them on its own. If you're coming over from Copilot, don't expect that auto-discovery behaviour to just be there - it isn't.

**Agents** diverge again. Copilot's headline agent feature is the **coding agent**: assign it a GitHub issue, it goes off and works in a cloud sandbox asynchronously, and eventually opens a PR. That's a different shape entirely from Claude's sub-agents, which run synchronously or in the background *within* a session, mostly for research and parallel digging rather than owning an issue end-to-end. Copilot's in-editor "agent mode" is closer to Claude's main agentic loop - multi-step, tool-using, one shared context throughout - than it is to Claude's sub-agent delegation.

---

## Final Verdict: How to Actually Use Them

- **Instructions** only get something if you'd be annoyed at its absence on *every single task* in the repo. Keep the file short - every line is a recurring tax.

- **Skills** get something if it's a real, repeated workflow, just not one you need on every task. Spend the extra five minutes on the name and description - that's the whole mechanism the model has to find it by.

- **Agents** get the open-ended stuff - the kind of question that would otherwise fill your context with search noise, or work you can genuinely split up and run at the same time. Don't reach for one to answer something a single file read would settle.

My own setup ended up small on purpose: a short `CLAUDE.md` with only hard rules, a growing pile of skills for anything workflow-shaped, and agents kept for the "go figure out how this works across the whole codebase" questions that used to chew through half my context window before I bothered separating the three properly. In another post I will go into how I use agent orchistration to keep a long-running session alive without hitting the context wall, but for now, just remember: instructions, skills, and agents are not interchangeable. Use them correctly and you'll save tokens, time, and frustration.
