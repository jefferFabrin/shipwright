---
description: Open the PR via `gh pr create` after /sw-pr-prep is green. Uses pr-body.md. Always opens as a draft.
argument-hint: <ticket-id>
---

# `/sw-pr-create`

Input: $ARGUMENTS (ticket id)

## Steps

1. **Confirm `/sw-pr-prep` reported all-PASS** for this ticket recently (same commit — if the branch moved since, re-run prep first rather than trusting a stale result).
2. **Push the branch** (`git push -u origin sw/<ticket>` from the worktree).
3. **Open the PR as a draft**, always — `gh pr create --draft --title "<ticket>: <title>" --body-file <pr-body.md path>`. The human is expected to run the affected flow locally at least once before flipping it to ready; that manual run is the thing that actually catches "the AI never ran this for real," not a checkbox.
4. **Record the PR URL** in `.shipwright/<ticket>/flow.md` and `state.json` (if orchestrated).
5. **Report the URL** and remind the human it's a draft pending their own local run.

## What this command must never do

- Open a non-draft PR, ever, regardless of how clean the review was.
- Push directly to the base branch.
- Merge anything — merging is always a separate, explicit human action.
- Add an AI co-author trailer to the PR body or any commit it touches.

This is the last fully-autonomous step in the pipeline. `/sw-handoff` (writing the human-facing test checklist) can run before or after this, but merging is never automated by this plugin.
