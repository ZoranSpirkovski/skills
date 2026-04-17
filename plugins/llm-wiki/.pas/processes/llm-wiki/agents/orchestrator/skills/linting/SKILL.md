---
name: linting
description: Use when the user asks for a health check, says the wiki feels stale, or at a recurring cadence (weekly is a reasonable default). Runs six independent checks and reports findings with proposed fixes.
---

# Linting

Audit the wiki for decay — unresolved contradictions, orphans, stale claims, missing cross-refs, undocumented concepts, and data gaps a web search could close.

## Steps

Run these checks and report findings in a structured list. Each check is independent; for large wikis, dispatch them in parallel via fire-and-forget Agent subagents, one per check.

- **Contradictions** — pages with a `## Contradictions` section that has not been resolved.
- **Orphan pages** — pages with no inbound links from other wiki pages. Either link them in or delete them.
- **Stale claims** — claims cited from one old source that newer sources don't confirm or actively weaken.
- **Missing cross-refs** — a page mentions an entity by name but doesn't link to the entity's page.
- **Concepts without pages** — a term appears in three or more pages but has no page of its own. Promote it.
- **Data gaps worth a web search** — open questions the wiki hasn't answered that a web search could close.

For each finding, propose a specific fix. Do not fix silently — list the fix and let the user approve.

Append a log entry:
```
## [YYYY-MM-DD] lint | <summary>
```

## Output

```
Lint: <wiki name>

Contradictions (N): 
- entities/raft.md — resolved? says "leader-based" vs "leaderless" from two sources
  → Fix: re-read both sources, keep the correct one, move the other to a ## Historical section

Orphans (N):
- concepts/gossip-protocol.md — no inbound links
  → Fix: link from entities/swim.md and concepts/eventual-consistency.md

Stale claims (N):
- entities/postgres.md claim on MVCC cites a 2019 source; the 2024 source disagrees
  → Fix: update the claim, flag both

Missing cross-refs (N):
- sources/raft-vs-paxos-practical.md mentions "Leslie Lamport" but doesn't link [[entities/leslie-lamport]]
  → Fix: convert to wikilink

Concepts without pages (N):
- "leader election" appears in 4 pages, no concept page
  → Fix: promote to concepts/leader-election.md

Data gaps (N):
- entities/raft.md open question: "how does Raft compare to viewstamped replication?"
  → Fix: web search + file as synthesis

Approve fixes? (y / per-finding)

Log entry:
## [YYYY-MM-DD] lint | <findings summary>
```

## Common Mistakes

- **Fixing silently.** Always list the proposed fix and wait for user approval. Silent fixes erode trust in the lint report.
- **Conflating "no inbound links" with "bad page."** Some pages are terminal (synthesis pages, external-only entities). When in doubt, propose linking rather than deleting.
- **Running lint without reading the schema.** `SCHEMA.md` may add domain-specific lint rules — respect them.
- **Skipping the log entry.** Lint counts as an operation.
