---
description: Orchestrator for the full ticket lifecycle — spec -> [GATE_A] -> implement -> qa -> review -> (rework loop, risk-weighted cap) -> pr draft -> handoff. One mandatory human gate (spec approval, unless the trivial-auto-approve path applies); everything after runs autonomously. Owns .shipwright/<ticket>/{state.json, flow.md}. Supports `status` and `resume`.
argument-hint: <ticket-id> | <freeform description> | status [ticket-id] | resume <ticket-id>
---

# `/sw-flow`

You do not reimplement any stage's logic here — you compose the standalone commands (`/sw-spec`, `/sw-implement`, `/sw-qa`, `/sw-review`, `/sw-pr-prep`, `/sw-pr-create`, `/sw-handoff`) in sequence, and you own persistence (`state.json`, `flow.md`) that those commands don't maintain on their own.

Input: $ARGUMENTS

## Subcommands

- **`status [ticket-id]`** — read `state.json`, render a plain-language stage tracker (done / in-progress / not-started per stage, current blocker if any). No ticket-id: list all tickets with a run in progress. Read-only, does not advance anything.
- **`resume <ticket-id>`** — read `state.json.stage`, continue the pipeline from there. First check `schema_version` matches what this plugin writes; if not, stop and report the mismatch instead of guessing how to reinterpret an old file.
- Anything else — start or continue a run: an existing ticket id (Entry B) or a freeform description (Entry A, creates the ticket via `/sw-spec`'s own Entry-A path).

## Pipeline

```
SPEC -> GATE_A -> IMPLEMENT -> QA -> REVIEW -> PR_DRAFT -> HANDOFF -> DONE
                                 ^-------- rework (REVIEW -> IMPLEMENT) --------|
(BLOCKED reachable from any stage)
```

1. **SPEC.** Run `/sw-spec` (Entry A or B as resolved above). Write `state.json` with `stage: SPEC`, `schema_version: 1`, resolved `risk.critical_path` from the spec's impact survey, and `rework.cap` set from `config.rework_cap.critical_path` or `.default` accordingly.

2. **GATE_A.** If the spec is `Trivial: true` **and** `config.auto_approve_trivial: true` **and** `risk.critical_path == MISS` **and** no schema/migration/dependency change per the impact survey — auto-approve, set `gates.auto_approved: true`, log it plainly in `flow.md` (never silent). Otherwise, present the spec and **stop for a real human approval** — this is the one mandatory gate, never skipped, never inferred from silence. On approval, `gates.spec_approved: true`, `stage: IMPLEMENT`.

3. **IMPLEMENT.** Run `/sw-implement <ticket>`. If it escalates (critical-path HIT or self-reported low confidence), record `escalations[]` and re-run on opus per that command's own logic — this is not a human stop, just a model upgrade.

4. **QA.** Run `/sw-qa <ticket>`.
   - PASS -> `stage: REVIEW`.
   - FAIL -> this is a rework cycle. Check `rework.count` against `rework.cap` **before** incrementing:
     - Under cap: increment, route back to `/sw-implement` scoped to the specific FAIL items only, then re-run QA.
     - At cap: `stage: BLOCKED`, `blocked_reason: "rework cap hit"`. Stop for a human.
   - If QA reports the failure is really a spec defect (contradictory/infeasible AC), do not spend a rework cycle — `stage: BLOCKED`, `blocked_reason: "spec defect — needs re-spec"`. A human decides whether to re-run `/sw-spec` (the only way GATE_A reopens).

5. **REVIEW.** Run `/sw-review <ticket>`.
   - CLEAN -> `stage: PR_DRAFT`.
   - REWORK -> same rework-cap logic as step 4, routing the BLOCKING findings back to `/sw-implement`.
   - Spec-defect finding -> same `BLOCKED` handling as step 4.

6. **PR_DRAFT.** Run `/sw-pr-prep <ticket>`; if all-PASS, run `/sw-pr-create <ticket>`. If prep FAILs, that's not a rework cycle against the code necessarily — diagnose which check failed and route appropriately (a failed full-suite run that only surfaces now is a real regression -> rework; a `gh auth` failure is an environment issue -> `BLOCKED` for the human to fix, not the agent's to solve).

7. **HANDOFF.** Run `/sw-handoff <ticket>`. `stage: DONE`.

8. **(Optional) POST_MERGE.** Only part of this pipeline if `config.deploy_stage_enabled: true`. If so, insert after HANDOFF: prompt the human that the PR needs to actually be merged first (this plugin never merges), then offer `/sw-deploy <ticket>` once it is.

## What the orchestrator must never do

- Skip or auto-pass GATE_A outside the exact trivial-auto-approve conditions above.
- Let any stage push, open a non-draft PR, write to an external ticket tracker, or deploy before the appropriate gate/stage.
- Add an AI co-author trailer anywhere.
- Exceed the rework cap silently — hitting it always means `BLOCKED`, never a third silent attempt.
- Drive a critical-path change through anything but opus for implement/review.
- Resume a `state.json` whose `schema_version` doesn't match without flagging the mismatch first.

## Concurrency

Multiple `/sw-flow` runs — even two touching the same repo — are safe via git worktrees, not re-cloning: `.shipwright-worktrees/<ticket>`, branch `sw/<ticket>`. Git itself refuses to check out the same branch in two worktrees, which is a free collision guard. Dependencies install fresh per worktree; don't share them across concurrent runs. If two tickets need a local dev server/port, each bumps to the next free local port rather than colliding. Don't run history-rewriting git operations while any flow is active.
