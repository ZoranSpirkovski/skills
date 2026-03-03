---
name: content-audit
description: Use when auditing any written content for clarity, jargon, logical flow, and AI-signature patterns — triggers on "audit this", "review my writing", "check this content", content quality review requests, or before publishing/sending important content
---

# Content Audit

Multi-persona audit engine that stress-tests written content through dynamically generated reader personas. Works with any content type — articles, emails, docs, marketing copy, social posts.

## When to Use

- "Audit this", "review my writing", "check this content"
- Content quality review before publishing or sending
- Detecting AI-signature patterns in writing

## When NOT to Use

- Code review (use code-reviewer agent)
- Grammar/spelling only (suggest a grammar tool)
- Content generation from scratch

## Modes

```
User provides content
        │
        ▼
  Has workspace checklist?
   ┌────┴────┐
   No       Yes
   │         │
   ▼         ▼
 QUICK    WORKSPACE
 Mode      Mode
 (4 phases) (5 phases)
```

**Quick mode** (default): Runs phases 1–3 and 5. No setup required.
**Workspace mode**: Runs all 5 phases including checklist audit. Requires a `.content-audit/` directory with checklist files (see Workspace Setup below).

## Pre-Audit Analysis

Before auditing, analyze the content to generate **3 reader personas** along a spectrum:

1. **Casual Consumer** — reads for general interest, low domain knowledge, skims
2. **Domain Professional** — works in the field, expects precision, catches inaccuracies
3. **Complete Newcomer** — zero context, needs everything explained, easily lost

For each persona, define:

| Field | Purpose |
|-------|---------|
| Name, age, occupation | Ground the persona in reality |
| Knows | What they bring to the content |
| Doesn't know | Knowledge gaps that cause confusion |
| Reading context | Where/how they encounter this content |
| "So What?" trigger | What makes them stop reading |
| Jargon sensitivity | Low / Medium / High |
| Flow tolerance | How much backtracking they accept |

Generate personas that match the content's domain — a crypto article gets different personas than a cooking blog.

## Audit Execution

Run 5 phases sequentially. Each phase uses the 3 personas as test readers. See `references/audit-phases.md` for full procedural details.

| Phase | Name | Tests |
|-------|------|-------|
| 1 | So What Audit | Every sentence serves a purpose |
| 2 | Jargon Audit | Every term is comprehensible |
| 3 | Logical Flow Audit | Information follows cognitive patterns |
| 4 | Checklist Audit | Domain-specific rules (workspace only) |
| 5 | Editor Pattern Check | Detect AI-signature writing |

## Output

After all phases complete, produce a structured audit report (template in `references/audit-phases.md`). Then present a numbered change list and ask: **"Apply changes? (all / select by number / none)"**

Only modify the content after the user chooses. Never auto-apply.

## Workspace Setup

For repeat use on the same content type, users create a one-time workspace:

```
.content-audit/
└── checklists/
    └── my-checklist.md    ← copy from references/example-checklist.md
```

Each checklist file has `applies_to` frontmatter that routes it to matching content types. See `references/example-checklist.md` for the template.
