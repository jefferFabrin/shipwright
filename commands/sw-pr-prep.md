---
description: Run the pre-PR preflight for a ticket — base branch, build, lint, tests, scope check, critical-path check, gh auth. Reports what's missing; does NOT open the PR.
argument-hint: <ticket-id>
---

# `/sw-pr-prep`

Input: $ARGUMENTS (ticket id)

## Steps (each reported as PASS / FAIL, not silently skipped)

1. **Review verdict is CLEAN.** If not, stop — this ticket isn't ready for a PR yet.
2. **Base branch is correct and current** — the branch is based on the resolved base from the spec, and rebasing/merging the latest base doesn't produce new conflicts.
3. **Build passes** on the worktree's current commit.
4. **Lint passes.**
5. **Full test suite passes** (not just the affected subset) — a final full-suite run is the last line of defense before the diff leaves your machine.
6. **Scope check** — `git diff <base>...HEAD --name-only` matches what `impl-notes.md` claims was touched; no surprise files.
7. **Critical-path re-check** (`/sw-critical-impact` against the actual final diff). If HIT, confirm every item in `guardrails.md`'s extra ritual is present in the artifacts (evidence, opus-authored review, rollback plan, rollout/watch plan) — missing any of these is a FAIL here, not something to defer to the human.
8. **`gh auth status`** succeeds and the target remote/repo is reachable — no point building a PR body if `gh pr create` will fail on auth.
9. **Assemble `pr-body.md`** (not yet opened) from `spec.md` (Context/Expected behavior/AC), `qa-report.md` (evidence recap), and `review.md` (verdict) — following whatever PR template the project already has (`.github/PULL_REQUEST_TEMPLATE.md` etc.) if one exists, otherwise a sensible default: Summary, Changes, Testing, Risk.

## Result

All PASS -> ready for `/sw-pr-create`. Any FAIL -> report exactly what's missing; do not attempt to fix it yourself (that's `/sw-implement`'s job) or open a draft PR anyway to "make progress."
