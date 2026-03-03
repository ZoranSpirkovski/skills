# Example Checklist Template

Copy this file to `.content-audit/checklists/` and customize for your domain.

## Frontmatter

Every checklist file starts with YAML frontmatter that controls routing:

```yaml
---
name: my-domain-checklist
applies_to: [blog-post, article, newsletter]
---
```

The `applies_to` field determines when this checklist runs. During audit, the agent identifies the content type and loads matching checklists. Use broad types (`article`, `email`, `social-post`, `documentation`, `marketing-copy`) or specific ones (`api-docs`, `changelog`, `press-release`).

## Checklist Format

Organize rules into categories. Each category gets a pass/fail verdict based on its rules.

---

## Example: Link Requirements

**Applies when content contains hyperlinks.**

- [ ] Every link has descriptive anchor text (no "click here" or "read more")
- [ ] No broken or placeholder links (no `#`, `TODO`, or `example.com`)
- [ ] External links open context the reader needs (not tangential references)
- [ ] No more than 2 links per paragraph (avoids link dumps)
- [ ] First mention of a key entity links to its canonical source

**Pass criteria:** All rules satisfied.
**Fail criteria:** Any rule violated. Log each violation with the offending text.

---

## Example: Length and Structure

**Applies to all long-form content (>500 words).**

- [ ] Has a clear opening that states what the reader will learn or gain
- [ ] Sections are 150–300 words each (shorter for web, longer for whitepapers)
- [ ] Every section has a subheading that summarizes its content
- [ ] No single paragraph exceeds 4 sentences
- [ ] Conclusion restates the key takeaway without introducing new information

**Pass criteria:** All structural rules met.
**Fail criteria:** Any structural violation. Log section/paragraph number and issue.

---

## Example: Formatting Rules

**Applies to content published on web platforms.**

- [ ] Bullet lists have 3–7 items (fewer = use prose, more = split into groups)
- [ ] Bold text is used for key terms on first introduction only (not for emphasis)
- [ ] Numbers under 10 are spelled out, 10+ use digits (except in data-heavy sections)
- [ ] Acronyms are expanded on first use
- [ ] Code or technical terms use inline code formatting

**Pass criteria:** Formatting is consistent throughout.
**Fail criteria:** Inconsistency found. Log location and specific rule broken.

---

## Writing Your Own Categories

Follow this pattern:

```markdown
## Category Name

**Applies when [condition].**

- [ ] Rule 1 — clear, testable statement
- [ ] Rule 2 — one rule per checkbox
- [ ] Rule 3 — specific enough to produce a yes/no answer

**Pass criteria:** What "good" looks like.
**Fail criteria:** What triggers a violation and what to log.
```

Keep rules testable. "Writing should be engaging" is not testable. "Every paragraph opens with a concrete statement, not an abstraction" is testable.
