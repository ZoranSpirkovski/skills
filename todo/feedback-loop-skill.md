# Feedback Loop Skill — Future Implementation

Companion skill to content-audit. Captures audit signals over time and uses them to evolve checklists and audit behavior. Build this after content-audit is battle-tested.

## Signal Types

Three categories of feedback signals collected during audits:

### PPU — Per-Persona Update

A persona consistently flags (or misses) the same issue type across multiple audits. Indicates the persona definition needs adjustment.

**Format:**
```yaml
signal: PPU
persona: [persona name]
pattern: [what they keep flagging/missing]
frequency: [how many audits]
suggested_change: [adjustment to persona fields]
```

### OQI — Output Quality Indicator

The user's reaction to audit suggestions — accepted, rejected, or modified. Tracks which audit phases produce useful vs. noisy output.

**Format:**
```yaml
signal: OQI
phase: [1-5]
suggestion_id: [reference]
user_action: accepted | rejected | modified
modification: [if modified, what changed]
```

### GATE — General Audit Tuning Event

Catch-all for signals that don't fit PPU or OQI. Examples: user adds a new banned word, user says "stop flagging X", user changes severity thresholds.

**Format:**
```yaml
signal: GATE
category: [banned-words | severity | scope | other]
description: [what changed]
source: [user instruction or pattern detection]
```

## Signal Documentation

Signals are stored in the workspace:

```
.content-audit/
├── checklists/
└── signals/
    ├── ppu-log.yaml
    ├── oqi-log.yaml
    └── gate-log.yaml
```

Each log is append-only during audits. The delta application workflow processes them periodically.

## Delta Application — 5-Phase Workflow

When enough signals accumulate, the feedback loop processes them:

### Phase 1: Signal Aggregation
- Read all signal logs
- Group by type (PPU, OQI, GATE)
- Count frequencies and identify patterns
- Threshold: process when 10+ signals accumulated since last delta

### Phase 2: Pattern Extraction
- PPU signals: identify persona fields that need updating
- OQI signals: calculate acceptance rate per phase (target: 70%+)
- GATE signals: extract actionable rule changes
- Output: list of proposed changes with confidence scores

### Phase 3: Conflict Resolution
- Check proposed changes against each other for contradictions
- Rules:
  - User explicit instruction (GATE) overrides pattern detection (PPU/OQI)
  - Higher frequency signals override lower frequency
  - Recent signals (last 5 audits) weighted 2x vs. older signals
  - If two changes contradict with equal weight, flag for user decision
- Output: resolved change list

### Phase 4: Application
- Apply changes to relevant files:
  - Persona adjustments → update default persona generation logic
  - Checklist changes → update or create checklist rules
  - Banned word changes → update editor-patterns.md
  - Severity changes → update verdict thresholds in audit-phases.md
- Each change gets a comment with the signal(s) that triggered it

### Phase 5: Verification
- Run a mini-audit on the last audited content with the new settings
- Compare results to previous audit
- Flag any changes that produce significantly different results (>20% change in findings)
- Present delta summary to user for approval

## Bloat Prevention

Checklists and persona definitions grow over time. Prevent runaway complexity:

- **Rule sunset:** If a checklist rule hasn't triggered a finding in 20+ audits, flag it for removal
- **Persona simplification:** If two personas consistently agree on 90%+ of findings, suggest merging
- **Signal pruning:** Archive signals older than 50 audits
- **Checklist size cap:** Warn when any single checklist exceeds 30 rules

## Quality Improvement Threshold Tests

Before applying a delta, verify it actually improves output quality:

1. **Noise reduction test:** Does the change reduce false positives? (Measure: rejection rate of suggestions should decrease)
2. **Coverage test:** Does the change maintain coverage? (Measure: acceptance rate shouldn't drop below 60%)
3. **Specificity test:** Are suggestions more actionable? (Measure: modification rate should decrease — users accept or reject cleanly, not rewrite)

If a delta fails any threshold test, log it but don't apply. Surface it to the user with the test results.
