---
description: After the PR is open, write a plain-language test checklist derived from the Acceptance Criteria — to .shipwright/<ticket>/handoff.md by default, or via the configured ticket adapter if one posts comments (GitHub Issues, Linear).
argument-hint: <ticket-id>
---

# `/sw-handoff`

Input: $ARGUMENTS (ticket id)

## Steps

1. **Load** `spec.md` (for the AC list and plain-language context), `qa-report.md`, `review.md`, and the PR URL from `state.json`/`flow.md`.
2. **Write `.shipwright/<ticket>/handoff.md`** using `references/handoff-template.md`: what changed (plain language), numbered how-to-test-in-the-app steps with one checkbox per AC, an honest "can't be checked through the app" section, the emitted (not run) regression-suite command if one applies, and a recap of what's already been checked automatically.
3. **If the PR exists**, append the same content as a "Testing" section on the PR description (via `gh pr edit --body-file` or equivalent) so it's visible without opening a second file.
4. **If `ticket_adapter` isn't `local`**, also post via that adapter's comment mechanism — but the file at `.shipwright/<ticket>/handoff.md` is always written regardless, as the source of truth.

This is the final autonomous step. It does not merge, does not deploy, and does not mark anything other than this ticket's own `stage: DONE`.
