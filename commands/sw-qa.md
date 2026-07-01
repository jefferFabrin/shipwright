---
description: Autonomous QA stage — spawns the sw-qa subagent (sonnet, read-only) to build/lint/test, run the spec's smoke test recipe, mutation-check new tests, and scan for test-tampering. Writes qa-report.md. Never fixes failures or opens a PR.
argument-hint: <ticket-id>
---

# `/sw-qa`

Input: $ARGUMENTS (ticket id)

## Steps

1. **Load `.shipwright/<ticket>/spec.md` and `impl-notes.md`.** If `impl-notes.md` is missing, `/sw-implement` hasn't run — stop and say so.
2. **Spawn the `sw-qa` subagent** in the same worktree the implementer used (not a fresh checkout — you want the actual committed change, uncommitted local state and all).
3. **On completion**, read the `Result` line in `qa-report.md`:
   - **PASS** -> report ready for `/sw-review`.
   - **FAIL** -> report the specific failing items (not the whole report) as what needs to change, and that this counts as a rework cycle if running under `/sw-flow`.

## What this command must never do

- Fix a failure it (or the subagent) finds — QA only ever reports.
- Skip the mutation check because "the new tests look obviously fine."
- Accept a pre-existing-test change without the exact `decisions.md` justification shape.
- Open a PR, push, update the ticket tracker, or deploy.

Called standalone, this writes only `qa-report.md`.
