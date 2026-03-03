# Audit Phases — Procedural Reference

Detailed instructions for each of the 5 audit phases. The agent runs these sequentially, using the 3 personas generated during pre-audit analysis.

---

## Phase 1: So What Audit

Every sentence must serve a purpose that at least one persona cares about. If no persona can answer "So what?" the sentence fails.

### Method

For each sentence in the content:

1. Present the sentence to each persona
2. Each persona asks: **"So what? Why should I care?"**
3. If the persona can articulate why this sentence matters to them → PASS
4. If the persona struggles or the answer is vague → WEAK
5. If no persona can justify the sentence → FAIL

### Verdict Definitions

| Verdict | Meaning |
|---------|---------|
| PASS | At least 2 personas find clear value |
| WEAK | Only 1 persona finds value, or value is indirect |
| FAIL | No persona can articulate why this sentence exists |

### Failure Categories

| Category | Description | Example |
|----------|-------------|---------|
| Orphan Fact | A fact with no connection to the argument | "The company was founded in 2015." (when founding date is irrelevant) |
| Empty Transition | A sentence that exists only to bridge paragraphs | "Let's now turn our attention to..." |
| Redundant Restatement | Repeats what the previous sentence said | "Revenue grew 20%. This represents a 20% increase." |
| Assumed Importance | States something is important without showing why | "This is a significant development." |
| Context-Free Comparison | Compares without a reference point | "This is faster than the previous version." (how much faster?) |

### Output Format

```markdown
### Phase 1: So What Audit

**Failures: [count] | Weak: [count] | Pass: [count]**

| # | Sentence | Type | Failed Personas | Revision |
|---|----------|------|-----------------|----------|
| 3 | "The company was founded in 2015." | Orphan Fact | All 3 | Delete or connect: "Founded in 2015, the company has spent [X] years..." |
```

---

## Phase 2: Jargon Audit

Every term must be comprehensible to the target audience. Terms that confuse any persona need fixing.

### Step 1: Term Extraction

Scan the content and extract all potentially problematic terms:
- Domain-specific terminology
- Acronyms and abbreviations
- Technical concepts
- Brand names or product names that aren't universally known
- Metaphors or idioms that may not translate

Generate domain-relevant jargon categories based on the content. For a tech article, categories might be: programming terms, infrastructure terms, business terms. For a cooking article: technique terms, ingredient terms, equipment terms. Do not use hardcoded categories.

### Step 2: Three-Persona Jargon Test

For each extracted term, test against all 3 personas:

| Verdict | Criteria |
|---------|----------|
| CLEAR | All 3 personas understand the term in context |
| NEEDS DEFINITION | 1–2 personas don't know the term; add inline definition |
| REPLACE | Term has a simpler alternative with no loss of meaning |
| REMOVE | Term adds jargon without adding meaning |

### The 5-Word Explanation Test

For any term marked NEEDS DEFINITION or REPLACE, attempt to explain it in 5 words or fewer. If you can, that explanation is the replacement. If you can't, the term may be necessary — add a brief inline definition instead.

### Output Format

```markdown
### Phase 2: Jargon Audit

**Violations: [count] | Clear: [count]**

| Term | Category | Verdict | Personas Confused | Fix |
|------|----------|---------|-------------------|-----|
| API endpoint | Technical | NEEDS DEFINITION | Newcomer | "API endpoint (the URL your app sends requests to)" |
| leverage | Business jargon | REPLACE | Newcomer, Consumer | → "use" |
```

---

## Phase 3: Logical Flow Audit

Content must follow cognitive patterns. Information should build on what came before, not jump around.

### Step 1: Entity/Topic Mapping

For each paragraph, list:
- Primary topic (what this paragraph is about)
- Entities introduced (people, concepts, products, terms)
- Entities referenced from earlier paragraphs
- New information vs. context-setting

### Step 2: Backtracking Detection

Scan the entity map for violations:

| Violation | Description | Detection |
|-----------|-------------|-----------|
| Entity Backtracking | Returns to a topic that was already closed | Entity appears in sections 2 and 5 but not 3 or 4 |
| Comparative Tangent | Introduces a comparison that derails the main argument | New entity appears only for comparison, then disappears |
| Premature Section Break | Breaks to a new section before the current topic is resolved | Key entity mentioned but not resolved before section boundary |
| Orphaned Details | Details that belong in an earlier section appear later | Entity's supporting details are separated from its introduction |
| Topic Fragmentation | A single topic is split across non-adjacent sections | Same primary topic in 2+ non-adjacent paragraphs |

### Step 3: Transition-by-Transition Audit

For each transition between paragraphs, check:

1. **Does the new paragraph follow logically?** (causal, temporal, or thematic link)
2. **Is the transition explicit or implicit?** (implicit is OK if the link is obvious)
3. **Does the reader need information they don't have yet?** (forward reference problem)
4. **Would a different paragraph order eliminate the need for backtracking?**

### Step 4: Three-Persona Flow Simulation

For transitions flagged in steps 2–3, simulate each persona reading through:

- Would the Casual Consumer get lost here?
- Would the Domain Professional find this ordering illogical?
- Would the Newcomer need to re-read?

### Step 5: Coherence Scoring

Score each section on a 5/3/1 scale:

| Score | Meaning |
|-------|---------|
| 5 | Clean flow — each paragraph builds on the last |
| 3 | Minor disruptions — reader pauses but recovers |
| 1 | Significant backtracking — reader must re-read to follow |

**Threshold:** All sections must score 3 or higher. Any section scoring 1 requires reorganization.

### Output Format

```markdown
### Phase 3: Logical Flow Audit

**Entity Map:**

| Section | Primary Topic | Entities Introduced | Entities Referenced |
|---------|---------------|--------------------|--------------------|
| 1 | [topic] | A, B | — |
| 2 | [topic] | C | A |

**Violations:**

| Location | Type | Description | Fix |
|----------|------|-------------|-----|
| §2→§3 | Entity Backtracking | Returns to entity A after closing it in §2 | Move §3 content into §2 |

**Coherence Scores:**

| Section | Score | Notes |
|---------|-------|-------|
| 1 | 5 | Clean |
| 2 | 3 | Minor tangent in ¶3, but recovers |

**Reorganization Plan:** (if any section scores 1)
[Proposed new section order with rationale]
```

---

## Phase 4: Checklist Audit (Workspace Mode Only)

Runs domain-specific checklists from the workspace `.content-audit/checklists/` directory.

### How Checklists Load

1. Agent identifies the content type (e.g., "blog-post", "email", "api-docs")
2. Scans all checklist files in `.content-audit/checklists/`
3. Loads files where `applies_to` matches the content type
4. If no domain checklists match, skip to the universal sub-checklist

### Universal Sub-Checklist

Always runs regardless of content type:

- [ ] **Banned AI words:** No words from the banned list in `references/editor-patterns.md`
- [ ] **Sentence length distribution:** 75%+ of sentences are under 20 words
- [ ] **Redundancy between sections:** No section substantially repeats another section's content
- [ ] **Spelling dialect consistency:** Content uses one dialect throughout (US or UK English, not mixed)

### Domain Checklist Evaluation

For each loaded checklist:
1. Evaluate every rule in every category
2. Mark each rule pass or fail
3. Log violations with the offending text and location
4. Calculate category-level verdict: all rules pass = PASS, any rule fails = FAIL

### Output Format

```markdown
### Phase 4: Checklist Audit

#### Universal Checklist

| Rule | Verdict | Notes |
|------|---------|-------|
| Banned AI words | FAIL | 3 found: "leverage" (¶2), "robust" (¶4), "crucial" (¶7) |
| Sentence length | PASS | 82% under 20 words |
| Redundancy | PASS | No cross-section redundancy |
| Dialect consistency | PASS | US English throughout |

#### [Domain Checklist Name]

| Category | Rules | Pass | Fail | Verdict |
|----------|-------|------|------|---------|
| Link Requirements | 5 | 4 | 1 | FAIL |
| Length and Structure | 5 | 5 | 0 | PASS |

**Violations:**

| Checklist | Category | Rule | Location | Issue |
|-----------|----------|------|----------|-------|
| [name] | Links | Descriptive anchor text | ¶3 | "click here" used as link text |

**Total violations: [count]**
```

---

## Phase 5: Editor Pattern Check

Scan for AI-signature writing patterns. See `references/editor-patterns.md` for the full pattern catalog and banned word list.

### Scanning Method

For each of the 6 patterns:
1. Scan the entire content for occurrences
2. Count instances
3. Check against the pattern's threshold
4. If over threshold, flag each instance with a revision

Also scan for all banned words and phrases from the banned list.

### Output Format

```markdown
### Phase 5: Editor Pattern Check

| Pattern | Count | Threshold | Verdict |
|---------|-------|-----------|---------|
| Endless Participles | 4 | 1/sentence | FAIL |
| Triple Enumerations | 5 | 2/piece | FAIL |
| Logical Flow Breaks | 1 | 0 | FAIL |
| Robotic Verb Choices | 3 | 0 | FAIL |
| Unnecessary Context | 2 | 0 | FAIL |
| Over-Specification | 1 | 0 | FAIL |

**Banned words found: [count]**

**Revisions:**

| # | Location | Pattern | Original | Revision |
|---|----------|---------|----------|----------|
| 1 | ¶2, S1 | Participles | "exploring while expanding and investing" | "explores new markets. It expands..." |
```

---

## Audit Report Template

After all phases complete, compile into this structure:

```markdown
# Content Audit Report

**Content:** [title or first line]
**Type:** [detected content type]
**Word count:** [count]
**Mode:** Quick / Workspace
**Date:** [date]

## Personas

### [Persona 1 Name] — [Role]
- Knows: [list]
- Doesn't know: [list]
- Reading context: [context]
- So What trigger: [trigger]

### [Persona 2 Name] — [Role]
[same fields]

### [Persona 3 Name] — [Role]
[same fields]

## Results Summary

| Phase | Findings | Verdict |
|-------|----------|---------|
| 1. So What | [X] failures, [Y] weak | PASS/FAIL |
| 2. Jargon | [X] violations | PASS/FAIL |
| 3. Flow | Lowest score: [X]/5 | PASS/FAIL |
| 4. Checklist | [X] violations | PASS/FAIL/SKIPPED |
| 5. Editor Patterns | [X] patterns found | PASS/FAIL |

**Overall: PASS / FAIL** (FAIL if any phase fails)

## Detailed Findings

[Phase 1 output]
[Phase 2 output]
[Phase 3 output]
[Phase 4 output]
[Phase 5 output]

## Change List

| # | Location | Change | Phase |
|---|----------|--------|-------|
| 1 | ¶2, S1 | Replace "leverage" → "use" | 2, 5 |
| 2 | ¶3 | Delete orphan fact about founding year | 1 |
| 3 | §2→§3 | Move ¶5 before ¶4 to fix backtracking | 3 |

**Apply changes? (all / select by number / none)**
```

## Verification Checklist

After generating the report, verify:

- [ ] All 3 personas are domain-appropriate (not generic)
- [ ] Phase 1 tested every sentence (not sampled)
- [ ] Phase 2 extracted terms from the actual content (not generic lists)
- [ ] Phase 3 entity map covers all sections
- [ ] Phase 4 loaded correct checklists (or was correctly skipped in quick mode)
- [ ] Phase 5 checked all 6 patterns and the banned word list
- [ ] Change list has no duplicates
- [ ] Every change in the list maps to a specific phase finding
- [ ] Report uses the template structure above
