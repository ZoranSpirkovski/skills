# Editor Patterns — AI-Signature Detection

Six writing patterns that signal AI-generated or AI-edited text. Scan content for each pattern, flag occurrences, and provide revisions.

## Pattern 1: Endless Participles

Stacking `-ing` words in a single sentence. Humans rarely chain more than one.

**Threshold:** 0–1 participle per sentence is natural. 2+ needs revision.

| Bad | Good |
|-----|------|
| "The company is **exploring** new markets while **expanding** its **existing** operations and **investing** in research" | "The company explores new markets. It expands operations and invests in research." |
| "**Running** the analysis while **monitoring** the results and **adjusting** parameters" | "Run the analysis. Monitor results and adjust parameters as needed." |

**Fix:** Split into shorter sentences. Replace participles with finite verbs.

## Pattern 2: Triple Enumerations

Listing exactly three items in a comma-separated group. AI defaults to three.

**Frequency rule:** 1–2 triple lists in a piece is fine. 3+ signals AI.

**Exception:** Factual lists that genuinely have three items (e.g., "red, green, blue") are fine. The pattern is suspicious when the grouping feels arbitrary.

| Bad | Good |
|-----|------|
| "This improves **speed, reliability, and scalability**" | "This improves speed and reliability" (if scalability isn't the point) |
| "Teams need **communication, collaboration, and coordination**" | "Teams need clear communication" (the other two are implied) |

**Fix:** Ask whether all items earn their place. Cut to two, or expand to a proper list if all matter.

## Pattern 3: Logical Flow Breaks

Consecutive sentences that don't connect. The reader must infer the relationship.

| Bad | Good |
|-----|------|
| "Revenue grew 15%. The team hired three engineers." | "Revenue grew 15%, funding three new engineering hires." |

**Fix:** Add an explicit connective or restructure so the relationship is obvious.

## Pattern 4: Robotic Verb Choices

Verbs that no human uses in casual speech. They sound authoritative but feel hollow.

| Replace | With |
|---------|------|
| leverage | use |
| utilize | use |
| facilitate | help, enable |
| implement | build, set up, add |
| optimize | improve |
| streamline | simplify |
| spearhead | lead |
| foster | encourage, support |
| bolster | strengthen |
| garner | get, earn |
| underscore | show |
| navigate | handle, deal with |
| pivot | shift, change |

**Principle:** If you wouldn't say it out loud to a colleague, replace it.

## Pattern 5: Unnecessary Context Insertions

Sentences that restate what the reader already knows, usually at paragraph openings.

**Detection:** Look for sentences that:
- Restate the article's topic ("In the world of X...")
- Recap what was just said ("As mentioned above...")
- Add context no one needs ("It's worth noting that...")
- Frame the obvious ("It's important to understand that...")

**Fix:** Delete the sentence. If the paragraph still makes sense, the context was unnecessary.

## Pattern 6: Over-Specification

Adding qualifiers, adjectives, or prepositional phrases that don't change meaning.

**Detection:** For each modifier, ask: "Does removing this change what the reader understands?" If no, delete it.

| Bad | Good |
|-----|------|
| "a **comprehensive and detailed** analysis of the **current** situation" | "an analysis of the situation" |
| "**actively** working to **significantly** improve" | "working to improve" |

**Fix:** Strip modifiers. Add back only those that genuinely change meaning.

---

## Banned AI Words

Words and phrases that signal AI authorship. Replace on sight.

### Verbs and Verb Phrases

| Banned | Replacement |
|--------|-------------|
| surge | rise, jump, increase |
| soar | climb, rise, grow |
| bolster | strengthen, support |
| underscore | show, reveal |
| spearhead | lead |
| navigate (metaphorical) | handle, manage |
| foster | encourage, build |
| garner | get, earn, attract |
| catapult | push, accelerate |
| grapple | deal with, face |
| pave the way | enable, lead to |
| cement (its position) | secure, establish |
| open the door to | allow, enable |
| set the stage for | prepare for, lead to |
| sends a clear signal | shows, indicates |
| demonstrates | shows |
| highlights | shows, reveals |
| signals | suggests, shows |
| underscores | shows |

### Adjectives and Adverbs

| Banned | Replacement |
|--------|-------------|
| crucial | important, key |
| pivotal | important, key |
| robust | strong, solid |
| comprehensive | full, complete, thorough |
| innovative | new, novel |
| significant | large, notable, big |
| unprecedented | new, rare, first |
| transformative | major, big |
| groundbreaking | new, first |
| cutting-edge | new, modern, latest |
| game-changing | major |
| essentially | (delete) |
| basically | (delete) |
| certainly | (delete) |
| indeed | (delete) |
| fundamentally | (delete or replace with context) |
| arguably | (delete or make the argument) |
| notably | (delete) |
| increasingly | (verify with data or delete) |
| landscape | space, market, field |
| paradigm | model, approach |
| ecosystem | system, network, market |
| synergy | (delete or describe the specific benefit) |

### Sentence Starters to Avoid

- "In today's rapidly evolving..."
- "It's worth noting that..."
- "It's important to understand..."
- "As we navigate..."
- "In the ever-changing landscape of..."
- "This is a testament to..."
- "At its core..."
- "When it comes to..."

### Superlatives Without Citation

Any superlative claim ("largest", "fastest", "most significant", "first-ever") must have a source. If no source exists, soften: "one of the largest", "among the fastest."
