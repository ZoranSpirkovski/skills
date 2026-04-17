---
name: querying
description: Use when the user asks a question that should be answered from the wiki. Reads candidate pages, synthesizes an answer with citations, and offers to file the answer back as a synthesis page so good answers compound instead of disappearing into chat history.
---

# Querying

Answer a user question from the accumulated wiki, with every claim cited, and offer to file the answer back.

## Steps

1. **Read `.wiki/wiki/index.md`** first to find candidate pages. At small scale (under ~100 sources), the index is enough; you do not need embedding search.
2. **Read the candidate pages in full.** Follow cross-references as needed. For long candidate lists, parallelize reads via fire-and-forget Agent subagents.
3. **Synthesize an answer with citations.** Every claim in your answer cites a wiki page slug so the user can verify, e.g. `Raft uses a leader-based approach [[entities/raft]]`.
4. **Offer to file the answer back.** Say: "This answer touches X, Y, and Z pages. Should I file it as a new synthesis page at `.wiki/wiki/synthesis/<slug>.md`?" Good answers should compound into the wiki — they should not disappear into chat history. If the user agrees, write the page with proper frontmatter, update the index under `## Synthesis`, and append a log entry with op `query`.

## Output

Answer text with one citation per claim, followed by:

```
Pages read:
- .wiki/wiki/entities/<slug>.md
- .wiki/wiki/concepts/<slug>.md
- ...

File back? Proposed: .wiki/wiki/synthesis/<slug>.md
```

If the user accepts the file-back:

```
Filed: .wiki/wiki/synthesis/<slug>.md
Index updated: +1 synthesis page

Log entry:
## [YYYY-MM-DD] query | <question summary>
- Filed: synthesis/<slug>.md
```

## Common Mistakes

- **Answering without citations.** Every claim must cite its source page. Without citations the answer is not verifiable and cannot be filed back meaningfully.
- **Not offering to file back.** The default is to offer — the user can decline. Skipping the offer is what makes wikis decay.
- **Reading only `index.md` excerpts instead of full pages.** Candidate pages must be read in full; the index is an entrypoint, not a knowledge source.
- **Writing the synthesis page without updating `index.md` and `log.md`.** Every file-back is a full ingest-shaped operation.
