---
description: Autonomous code-review stage — spawns the sw-reviewer subagent (opus, read-only) to review the diff for correctness, scope, critical-path ritual, cross-cutting drift, and independently re-checked test-tampering. Writes review.md with BLOCKING/NON-BLOCKING findings. Does not edit code or open a PR.
argument-hint: <ticket-id>
---

# `/sw-review`

Input: $ARGUMENTS (ticket id)

## Steps

1. **Load** `spec.md`, `impl-notes.md`, `qa-report.md`, `decisions.md` for the ticket. If `qa-report.md`'s `Result` isn't `PASS`, refuse — review only runs after QA passes.
2. **Spawn the `sw-reviewer` subagent** (always opus, not conditional) against the same worktree/branch.
3. **On completion**, read the `Verdict` line in `review.md`:
   - **CLEAN** -> report ready for `/sw-pr-prep`.
   - **REWORK** -> report the BLOCKING findings only as what `/sw-implement` needs to fix, and note this is a rework cycle if running under `/sw-flow`. If the reviewer flagged a finding as a spec defect rather than an implementation bug, say so explicitly — that routes to `BLOCKED` for a human, not a normal rework cycle.

## What this command must never do

- Run before QA has passed.
- Edit any file.
- Downgrade the reviewer off opus for any reason.
- Reclassify a real BLOCKING finding as NON-BLOCKING to avoid a rework cycle.
- Open a PR, push, update the ticket tracker, or deploy.

Called standalone, this writes only `review.md`.
