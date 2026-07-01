---
description: Initialize Shipwright in the current project — creates .shipwright/config.yml and guardrails.md, gitignores the run trail, and asks the handful of questions the pipeline needs answered once.
argument-hint: (no arguments)
---

# `/sw-init`

One-time setup for a project that just installed the shipwright plugin. Idempotent — safe to re-run; never overwrites an existing answer without confirming.

## Steps

1. **Check for an existing `.shipwright/config.yml`.** If present, report its current settings and ask only what's missing rather than starting over.

2. **Ask the human (batch these, don't trickle one at a time):**
   - Artifact language for generated docs (spec/flow/review/etc.) — default: match the language the human is using in this conversation.
   - Rework caps — accept the defaults (2 normal / 1 critical-path) unless they want different numbers.
   - `auto_approve_trivial` — off by default; explain what "trivial" means (single file, non-critical-path, no schema/migration/dependency change) before asking.
   - `deploy_stage_enabled` — off by default; only worth turning on if there's an actual deploy command/target to wire up right now.
   - Mutation tool — check the repo for a real mutation-testing tool config (Stryker `stryker.conf.*`, `mutmut.ini`, `pitest` in a build file, etc.). If found, set `mutation_tool` to it. If not, leave `auto` (falls back to the manual analytical check) and mention it's worth adding a real tool later for the primary language.

3. **Write `.shipwright/config.yml`** from `references/config-template.md`, filled with the answers above. `schema_version: 1`.

4. **Write `.shipwright/guardrails.md`** from `references/guardrails-template.md`. If you have visibility into the project's docs/specs (PRD, tech specs, ADRs, security docs), scan them for anything that reads like a sensitive/critical path — auth, tenant isolation, payments/billing, data-deletion, migrations, public unauthenticated endpoints, compliance (LGPD/GDPR/PCI), audit logging — and seed the table with a first draft instead of leaving it empty. Tell the human this is a first draft they own and should correct, not a finished audit.

5. **Create `.shipwright/tickets/` directory** (empty, `.gitkeep`).

6. **Gitignore the run trail, but not everything.** Add to `.gitignore`:
   ```
   .shipwright/*/
   !.shipwright/tickets/
   .shipwright-worktrees/
   ```
   This ignores per-ticket run directories (`state.json`, `qa-report.md`, etc. — ephemeral machine output) but keeps `config.yml`, `guardrails.md`, and `tickets/` tracked. If the human wants `flow.md`/`decisions.md` kept as a durable history per ticket (recommended), tell them to `git add -f .shipwright/<ticket>/flow.md .shipwright/<ticket>/decisions.md` when a ticket finishes — mirrors how the flow already treats those two files as worth keeping.

7. **Summarize** what was written and the one or two things most worth double-checking by hand (usually: the guardrails first draft, and whether `auto_approve_trivial` should really be on for a solo project).

Do not start a ticket or spec during init — this command only sets up the project, it doesn't do engineering work.
