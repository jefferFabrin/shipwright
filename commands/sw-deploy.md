---
description: Optional deploy stage for a merged ticket. Off by default (config.deploy_stage_enabled). Generic — asks for the project's actual deploy command/target rather than assuming a provider, and links the result back into the ticket's state so deploys aren't orphaned from the lifecycle that produced them.
argument-hint: <ticket-id> <env>
---

# `/sw-deploy`

Input: $ARGUMENTS (ticket id, environment name — e.g. `staging`, `prod`)

This command intentionally does not assume Vercel, AWS, Fly, or any other target — the original design this plugin improves on had deploy as a fully separate, hand-authored, single-repo command with no link back to the ticket that produced the change. Here, a deploy is still a distinct human-triggered action, but it's recorded against the ticket's own trail instead of floating free.

## Steps

1. **Confirm the ticket's PR is actually merged** (not just draft-open) — check via `gh pr view --json state,mergedAt`. If not merged, stop; this command deploys merged code only, never a branch.
2. **Look for a project-defined deploy recipe** — a `deploy` script in the project's task runner (package.json script, Makefile target, CI workflow dispatch, etc.). If found, confirm with the human which one applies to `<env>` and run it.
3. **If no recipe exists**, ask the human directly for the exact command/target rather than guessing a cloud provider's CLI incantation — this is exactly the kind of "affects shared systems" action that needs an explicit human-provided command, not an assumed default.
4. **Extra confirmation gate for any environment that looks like production** (`prod`, `production`, or whatever the human's answer implies) — restate what's about to run and against what target, and get an explicit go-ahead before executing.
5. **Record the result** in `.shipwright/<ticket>/flow.md` (timestamp, env, outcome) and, if `config.deploy_stage_enabled: true` and this is part of an orchestrated run, update `state.json` with a `POST_MERGE` entry.

## What this command must never do

- Deploy unmerged code.
- Guess a deploy target/command the human hasn't confirmed.
- Skip the extra confirmation for a production-looking environment.
- Roll back a deploy on its own initiative — a bad deploy is a human call, this command reports, it doesn't self-heal.
