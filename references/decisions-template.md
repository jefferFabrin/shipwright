# `.shipwright/<ticket>/decisions.md` template

ADR-lite. Every stage appends an entry for each meaningful autonomous decision — not routine work, the moments where a stage picked between real alternatives or deviated from the obvious path.

```markdown
## <YYYY-MM-DD HH:MM UTC> — <stage> — <short title>

**Decision:** what was chosen.
**Alternatives considered:** what else was on the table and why it lost.
**Rationale:** the actual reasoning — this is the part worth reading in six months.
```

## Canned shapes for the three recurring cases

**Escalation to Opus**
```markdown
## <ts> — <stage> — escalated to opus
**Trigger:** critical_path_hit | self_reported_low_confidence
**Decision:** re-spawn this stage on opus instead of sonnet.
**Rationale:** <matched glob / specific source of uncertainty — be concrete, "not sure" is not a rationale, name the actual ambiguity>
```

**Declined an out-of-scope change**
```markdown
## <ts> — implement — declined out-of-scope change
**Decision:** did not touch <file/thing>, even though <temptation, e.g. "it has an obvious unrelated bug">.
**Rationale:** outside this ticket's Out of scope boundary. Flagging here instead of fixing silently so it doesn't get lost.
```
Consider filing it as a follow-up ticket via `/sw-ticket` rather than letting it evaporate.

**Pre-existing test changed**
```markdown
## <ts> — implement — changed a pre-existing test
**Decision:** modified `<path>` because <contract actually changed per AC #n>.
**Rationale:** <why the old assertion no longer describes correct behavior>
**Strictness claim:** the new assertion is stricter-or-equal to the old one, specifically: <how>.
```
QA and review both key off this exact entry shape — a pre-existing test change without one of these blocks is BLOCKING by default.

## Self-reported low confidence (new in this project vs. the original design)

Any stage may flag itself uncertain about a real decision it just made, even without a critical-path glob match. This is what triggers the `self_reported_low_confidence` escalation trigger in config — use it honestly; it's cheaper than a subtle bug shipped confidently.

```markdown
## <ts> — <stage> — self-reported low confidence
**Uncertain about:** <the specific decision or assumption>
**Why:** <what's genuinely ambiguous — missing spec detail, conflicting existing code, a guess about undocumented behavior>
**Requesting:** escalation to opus for this stage | flag for human review at next gate (pick one)
```
