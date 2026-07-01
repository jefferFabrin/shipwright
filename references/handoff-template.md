# `.shipwright/<ticket>/handoff.md` template

Written by `/sw-handoff`, the final stage. On a solo project this is for *you*, ten minutes or ten weeks from now, deciding whether to actually click "merge" — it is not written for an audience that needs convincing, so keep it honest, including the "can't be checked through the app" section.

```markdown
# Ready to test: <ticket>

PR: <url> (draft — flip to ready once you've run the flow below yourself at least once)

## What changed
2-4 plain-language sentences. No jargon a non-technical stakeholder would trip on, even if the only stakeholder is future-you on a tired evening.

## How to test it (in the app)
1. <numbered UI/API steps>
2. ...
- [ ] AC #1 — <restated in plain language>
- [ ] AC #2 — ...

## Can't be checked through the app
Anything true but not observably testable via UI/API alone (an internal refactor, a log-format change, a performance improvement without a visible counter). Honest disclosure, not a place to hide gaps.

## Targeted regression test
If this project has an e2e/regression suite, the exact command to run it for the affected flow (emitted, never auto-run by `/sw-handoff`):
`<command>`

## Already checked automatically
Recap of what qa-report.md and review.md already confirmed (build/lint/tests/mutation-check/critical-path ritual), so you're not re-verifying work already done — you're doing the one thing the pipeline can't: judging it against what you actually wanted.
```

## Where this gets delivered

Default (`ticket_adapter: local`, no integrations configured): written to `.shipwright/<ticket>/handoff.md` and also printed to the PR description as a "Testing" section by `/sw-pr-create`. If you later configure a ticket adapter (GitHub Issues, Linear), `/sw-handoff` posts the same content there instead — the template doesn't change, only the delivery channel does.
