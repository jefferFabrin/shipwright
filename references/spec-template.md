# `.shipwright/<ticket>/spec.md` template

```markdown
# <Title>

Status: DRAFT               <!-- DRAFT -> APPROVED (human gate) -->
Entry: A                     <!-- A = greenfield description, B = existing local ticket file -->
Ticket file: .shipwright/tickets/<ticket>.md
Trivial: false               <!-- set true only if it qualifies for auto_approve_trivial; see config.yml -->

## Context
Why this exists. What triggered it (bug report, product ask, your own itch).

## Current behavior
What happens today, concretely — not "it's broken" but the exact observed behavior.

## Expected behavior
What should happen instead, concretely.

## Acceptance criteria
Given/When/Then, one per line, each independently testable.
- [ ] Given ..., when ..., then ...

## Out of scope
Explicit. Anything tempting-but-not-this-ticket goes here so the implementer
has a written boundary to point to instead of guessing. (This section is what
sw-reviewer checks scope drift against — an empty section is a bug in the spec,
not a sign there's nothing to exclude.)

## Technical approach
- Repo / base branch: <path>, base = <branch>
- Approach (3-6 bullets): the shape of the change, not a full design doc
- Test plan: what you'll add, at what level (unit/integration/e2e)

## Smoke test recipe
The declarative, mechanically-followable steps QA will run to confirm the
change actually works end-to-end — not left to QA's judgment call. Each step
names an exact command or UI action and the expected observable result.
1. Run `<command>` -> expect `<output/exit code>`
2. Hit `<route/screen>` as `<role/state>` -> expect `<observable result>`
3. (repeat as needed)

## Impact survey
- Critical-path: HIT / WARN / MISS (from /sw-critical-impact) — matched globs: [...]
- Cross-cutting module touched (e.g. a shared package multiple apps depend on): yes/no — which one
- Schema/migration involved: yes/no
- Third-party integration touched (billing, auth, email, AI provider, etc.): yes/no — which
```

## Notes

- The 5 canonical sections (Context, Current behavior, Expected behavior, Acceptance criteria, Out of scope) are mandatory; `Technical approach`, `Smoke test recipe`, and `Impact survey` are what the implementer/QA/reviewer stages actually key off of.
- `Trivial: true` is a claim the spec author makes, not something `/sw-spec` decides unilaterally — `/sw-flow` still double-checks it against `config.auto_approve_trivial` and the impact survey before skipping GATE_A. A false claim (e.g. marking a migration "trivial") is caught at that check, not trusted blindly.
- If, during implementation or review, the *spec itself* turns out to be wrong (contradictory AC, infeasible approach) — that is never treated as a rework cycle against the code. Stop, set `stage: BLOCKED`, `blocked_reason: "spec defect — needs re-spec"`, and let a human decide whether to re-run `/sw-spec`.
