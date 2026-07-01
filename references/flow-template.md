# `.shipwright/<ticket>/flow.md` template

The single "read this and understand everything" artifact for a run. Owned and appended to only by `/sw-flow`.

```markdown
# Flow: <ticket> — <title>

Entry: A | B
Repo / branch / worktree: <repo> / <branch> / <worktree path>
Outcome: IN PROGRESS | DONE | BLOCKED (<reason>)

## Timeline
| UTC timestamp | Stage | Model | Result | Notes |
|---|---|---|---|---|
| 2026-06-30T17:40:00Z | SPEC | sonnet | drafted | awaiting GATE_A |
| 2026-06-30T17:55:00Z | GATE_A | — | approved by you | |
| 2026-06-30T18:00:00Z | IMPLEMENT | sonnet | committed | 3 files, 2 commits |
| ... | | | | |

## Escalations
- (stage, trigger, reason, model change) — empty if none

## Rework cycles
Count: 0 / cap <cap resolved from config + risk>
- (cycle #, triggered from which stage, one-line reason) — empty if none

## Links
- Ticket file: .shipwright/tickets/<ticket>.md
- Branch: <branch>
- PR: <url or "not yet opened">
- Artifacts: spec.md, impl-notes.md, qa-report.md, review.md, decisions.md, pr-body.md, handoff.md
```

Keep this file (and `decisions.md`) out of `.gitignore` even though the rest of `.shipwright/<ticket>/` is typically ignored — these two are worth committing (`git add -f`) as a durable record of *why*, especially on a solo project where you're your own institutional memory six months later.
