---
name: context-file-hygiene
description: Use when creating, auditing, or maintaining CLAUDE.md or agent MD files — after /init, when context files feel bloated, when agents misbehave despite instructions, or when reviewing what belongs in a context file
---

# Context File Hygiene

## Core Principle

**If the agent can discover it from the codebase, it does not belong in the context file.**

Context files cost tokens on every request, bias agents toward mentioned patterns even when irrelevant, and go stale silently. Research shows LLM-generated context files decrease performance by 3% on average and increase costs by 20%+. Developer-written ones improve performance by only 4%.

**The burden of proof is on keeping, not removing.** Default action is REMOVE. You must justify every line that stays.

## When Invoked

1. Find the project's context file (CLAUDE.md, agent.md, .cursorrules, or similar)
2. Read the codebase to verify what's discoverable (Makefile, package.json, Gemfile, config files, directory structure)
3. Evaluate each section using the decision framework below
4. Present findings table to the user
5. Apply changes after approval

**Audit only.** Evaluate and trim what exists. Do not invent new directives during an audit — that's the user's job based on observed agent failures.

## Decision Framework

For each item, ask these questions in order:

**Q1: Can the agent find this by reading config files or globbing the codebase?**
- Package dependencies, stack description, architecture overview → REMOVE
- File structure, directory layout, available commands → REMOVE
- Model/schema relationships, route definitions → REMOVE
- Available components, controllers, services → REMOVE

**Q2: Does this correct a specific behavior the agent consistently gets wrong?**
- If YES and you've observed the failure repeatedly → KEEP, word as a short directive
- If NO or you're guessing it might help → REMOVE

**Q3: Is this a constraint the codebase cannot express?**
- "Don't modify backend during UI tasks" → KEEP (no file enforces this)
- "Run commands in Docker, never on host" → KEEP (Makefile doesn't say this)
- "Use Minitest not RSpec for new tests" → KEEP (both exist, agent can't guess preference)

Everything else: REMOVE.

## What Belongs in a Context File

- Environment constraints agents cannot infer ("all commands run in Docker")
- Lane restrictions ("don't modify backend during frontend tasks")
- Preferences when multiple valid options exist ("use Minitest for new tests")
- Patterns that contradict model training ("we use X instead of conventional Y")
- Pointers to non-obvious files the agent should read first ("read .interface-design/system.md before any UI work")

## What Does NOT Belong

- Project descriptions (README exists, agent can read it)
- Architecture overviews (agent explores the codebase in seconds)
- Dependency lists (package.json, Gemfile are right there)
- Route namespaces (agent runs route commands or reads config/routes.rb)
- Model relationships (agent reads model files and schema)
- Available commands (agent reads Makefile/package.json)
- Lists of components, controllers, or services (agent globs the directories)
- Authentication setup (agent reads the code)
- Counts of anything ("42 components" — goes stale immediately)

## Rationalizations to Reject

| Excuse to keep something | Reality |
|---|---|
| "Saves exploration time" | Agents explore in seconds. The 20%+ cost increase from bloated context outweighs time saved. |
| "Useful as a quick reminder" | Reminders bias the agent toward mentioned patterns even when irrelevant. |
| "Non-obvious" | If you found it by reading one file, it's obvious to the agent too. |
| "Concise and accurate" | Well-written descriptions are still unnecessary if discoverable. |
| "Provides orientation" | Agents orient themselves. That's what tool use is for. |
| "Prevents mistakes" | Only keep if you've observed the mistake repeatedly. Hypothetical prevention adds noise. |

## Maintenance

- **On new model releases:** Delete the context file entirely. Add back only what the agent gets wrong.
- **When agents misbehave:** Fix the codebase first (better tests, clearer structure, renamed files). Context file is the last resort.
- **The file should shrink over time, never grow.** Each model release makes agents better at discovery.

## Output Format

Present findings as:

| Section | Verdict | Reasoning |
|---|---|---|
| Section name | KEEP / REMOVE / REWORD | Why, referencing the decision framework |

Then present the trimmed file. A well-maintained context file is typically 5-15 lines of behavioral directives.
