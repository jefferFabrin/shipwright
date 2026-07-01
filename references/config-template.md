# `.shipwright/config.yml` — project configuration

Every project that installs this plugin gets one config file at its repo root, created by `/sw-init` (or by hand). All Shipwright commands read it before doing anything project-specific. Nothing here is hardcoded into the commands themselves — that's the point.

```yaml
schema_version: 1

# Ticket source of truth. "local" needs nothing external: tickets are markdown
# files under .shipwright/tickets/<id>.md. Swap in an adapter later without
# touching the pipeline commands — they only ever call "read ticket" / "write ticket".
ticket_adapter: local              # local | github-issues | linear (only "local" ships in v0.1)

# Output language for generated artifacts (spec.md, flow.md, review.md, etc).
# Code, comments, and commit messages always follow the project's own convention,
# not this setting.
artifact_language: pt-BR

# Model policy. Default is the cheap model everywhere; escalation is opt-in per trigger.
models:
  default: sonnet
  escalate_to: opus
  escalation_triggers:
    - critical_path_hit             # /sw-critical-impact reports HIT on files this stage will touch
    - self_reported_low_confidence  # the stage agent itself flags genuine uncertainty (see references/decisions-template.md)

# Rework budget before a stage gives up and blocks for a human. Risk-weighted:
# a critical-path change gets fewer autonomous retries, not more, because a second
# guess on a sensitive path is a worse failure mode than stopping early.
rework_cap:
  default: 2
  critical_path: 1

# Whether a DRAFT spec can skip the human GATE_A click-through. Only applies when
# ALL of these hold: single file touched, risk.critical_path == "MISS", no schema/
# migration/dependency-manifest change, ticket explicitly marked "trivial: true"
# by /sw-spec. Off by default — turn on once you trust the loop on your project.
auto_approve_trivial: false

# Optional stage. Off by default. When on, /sw-flow adds POST_MERGE after PR_DRAFT
# and /sw-deploy becomes part of the driven pipeline instead of a standalone command.
deploy_stage_enabled: false

# Mutation-testing tool to try before falling back to the manual analytical check
# (revert the fix, confirm the new test fails). "auto" detects by project files
# (stryker config -> JS/TS, mutmut/cosmic-ray -> Python, pitest -> JVM, etc).
mutation_tool: auto

# Who approves GATE_A / unblocks BLOCKED runs. Free text, purely informational —
# shown in flow.md and status output. Not an access-control mechanism.
approvers:
  - you
```

## Adapter contract (for `ticket_adapter`)

Every adapter must implement exactly two operations, used by `/sw-spec` and `/sw-flow`:

- **read(ticket_id) -> { title, body, status }** — fetch or load a ticket's current content.
- **write(ticket_id, { title, body, status })** — create or update it.

`local` implements this against `.shipwright/tickets/<id>.md` (frontmatter `title`/`status` + a markdown body). A future `github-issues` adapter would implement the same two calls against `gh issue view` / `gh issue edit`; a future `linear` adapter against the Linear MCP tools. Commands never branch on the adapter — they call `read`/`write` and the adapter resolves what that means.
