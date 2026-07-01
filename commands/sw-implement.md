---
description: Autonomous implementation stage — spawns the sw-implementer subagent (sonnet, escalates to opus on critical-path HIT or self-reported low confidence) in an isolated git worktree to implement an approved spec. Refuses without an APPROVED spec. Writes code + impl-notes.md; never opens a PR or pushes.
argument-hint: <ticket-id>
---

# `/sw-implement`

Input: $ARGUMENTS (ticket id)

## Steps

1. **Load `.shipwright/<ticket>/spec.md`.** If missing or `Status:` isn't `APPROVED`, **refuse** — report what's missing and stop. Never implement against a draft.
2. **Resolve the base branch** from the spec's Technical approach. If unresolved, stop and ask the human — do not guess.
3. **Create the isolated worktree**: `git worktree add ../.shipwright-worktrees/<ticket> -b sw/<ticket> origin/<base>` (or local `<base>` if there's no remote yet — common on a brand-new repo). Copy any gitignored `.env*` files into the worktree (they aren't tracked, so a worktree doesn't get them for free) — preserve nested paths in a monorepo. Install dependencies fresh in the worktree; never share `node_modules`/equivalent across concurrent worktrees.
4. **Spawn the `sw-implementer` subagent** with: the ticket id, the spec path, the worktree path, and the resolved base branch. Sonnet by default; if `.shipwright/config.yml`'s escalation triggers fire (critical-path HIT or the implementer self-reports low confidence), the subagent aborts and you re-spawn it on opus — record the escalation in `state.json.escalations` (if orchestrated) and `decisions.md`.
5. **On completion**, confirm `impl-notes.md` and `decisions.md` exist and are non-empty, and that at least one commit landed on the branch. Report a compact summary: branch, files touched, commits, build/lint/test evidence, any escalation.

## What this command must never do

- Run without an approved spec.
- Implement in the primary checkout instead of a worktree.
- Open a PR, push, update the ticket tracker, or deploy — that's not this stage's job even though the subagent has `Bash`.
- Silently continue on sonnet after a critical-path HIT or a self-reported low-confidence flag.

Called standalone, this writes only `impl-notes.md`/`decisions.md` — `flow.md`/`state.json` are only maintained by an orchestrated `/sw-flow` run.
