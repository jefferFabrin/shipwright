# Setup

## Install the plugin

This repo is a Claude Code plugin (see `.claude-plugin/plugin.json`). From any project where you want to use it:

1. Add this repo as a plugin source and install it, following Claude Code's plugin install flow (`claude plugin install <path-or-url>` or via a marketplace entry pointing at this repo, once it's pushed to GitHub).
2. Confirm it loaded: the `sw-*` commands and the `sw-implementer`/`sw-qa`/`sw-reviewer` agents should be available in that project.

## Per-project setup

Run once per project:

```
/sw-init
```

This writes, at the project's repo root:

- `.shipwright/config.yml` — model policy, rework caps, language, whether the trivial-auto-approve path and the optional deploy stage are on.
- `.shipwright/guardrails.md` — the critical-path glob table. `/sw-init` will draft a first pass from the project's own docs if it has any (PRD, tech specs, ADRs) — **read it and correct it**, it's a starting point, not an audit.
- `.shipwright/tickets/` — where local tickets live if you're using the default `local` ticket adapter.
- Updates `.gitignore` so per-run machine output (`state.json`, `qa-report.md`, etc.) doesn't get committed, while `config.yml`, `guardrails.md`, and `tickets/` stay tracked.

## Day to day

```
/sw-ticket "<short description>"     # shape a rough idea into a ticket
/sw-spec <ticket-id>                 # or skip straight to this — it creates the ticket too
/sw-flow <ticket-id>                 # run the full pipeline; stops once for your spec approval
/sw-flow status                      # see what's in flight
/sw-flow resume <ticket-id>          # pick a run back up after a BLOCKED stop
```

Each stage command (`/sw-implement`, `/sw-qa`, `/sw-review`, `/sw-pr-prep`, `/sw-pr-create`, `/sw-handoff`) also runs standalone if you want to drive the pipeline by hand instead of letting `/sw-flow` compose it for you — useful the first few times, while you're still building trust in the loop.

## Things worth deciding early, per project

- **`artifact_language`** — the language generated docs (spec/flow/review/etc.) are written in. Code and commit messages always follow the project's own existing convention instead, regardless of this setting.
- **Mutation testing tool** — if the project's primary language has a real mutation-testing tool (Stryker for JS/TS, mutmut for Python, PIT for JVM), wire it up and point `config.yml`'s `mutation_tool` at it. Otherwise QA falls back to a manual revert-and-confirm-fail check, which works but is slower and more conservative.
- **`auto_approve_trivial`** — leave off until you've run a few tickets through by hand and trust what "trivial" catches in practice for this specific project.
