# `.shipwright/<ticket>/review.md` template

Written by `sw-reviewer` (always Opus). Read-only, adversarial pass. Never trusts an earlier stage's claim without re-checking it.

```markdown
# Review: <ticket>

Verdict: CLEAN / REWORK
Rework iteration: <n> / <cap resolved from config + risk>

## Findings
| # | Severity | File:line | Finding | Dimension |
|---|---|---|---|---|
```

## The 6 dimensions (every finding must be classified into exactly one)

1. **Critical-path ritual** — independently re-run `/sw-critical-impact` against the diff, don't trust `impl-notes.md`'s claim. HIT without the full extra ritual (see `guardrails.md`) = BLOCKING.
2. **Scope vs. spec's Out of scope** — any file, import, module, or env var touched outside the spec's stated scope = BLOCKING, no matter how small or well-intentioned.
3. **Test tampering** — independently re-run the anti-tampering diff and independently re-verify the mutation-check claim, especially on any file that also matches the critical-path table. Do not accept QA's PASS at face value here.
4. **Cross-cutting drift** — if this project has a known "canonical source + N mirrors" pattern (see `guardrails.md` if one is documented for this project), confirm all copies moved together. Only one copy changing = BLOCKING with a note on which sibling location needs the same edit.
5. **Correctness** — actual bug hunting: off-by-one, wrong boundary/exclude logic, null/undefined handling, contract mismatches between caller and callee, race conditions in async paths.
6. **Style / maintainability** — always NON-BLOCKING, *unless* the style problem is severe enough to actually cause a correctness or critical-path issue, in which case reclassify it under dimension 1 or 5 and it inherits that severity.

## Verdict routing

- **CLEAN** — every dimension checked, no BLOCKING findings (NON-BLOCKING findings are fine to note and move on) -> proceed to `/sw-pr-prep`.
- **REWORK** — one or more BLOCKING findings -> route back to `/sw-implement`, scoped to just the BLOCKING findings, never a full re-implementation. Increment `rework.count` in `state.json`.
- If a BLOCKING finding is actually a flaw in the *spec* itself (contradictory or infeasible AC) rather than the implementation, do not spend a rework cycle on it — report it as a spec defect and let `/sw-flow` set `stage: BLOCKED`.
