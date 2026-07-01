# shipwright

A Claude Code plugin that orchestrates an end-to-end engineering pipeline — spec → implement → QA → review → draft PR — for one person working across their own projects. No ticket tracker, no team, no shared infra required.

It's a generalized, personal descendant of a larger internal system built for a real engineering team ([palazzo-engineering](#relationship-to-palazzo-engineering)), stripped of the assumptions that only make sense with a team (Linear as source of truth, named human approvers, a fixed post-mortem-driven glob table) and hardened in a few places that system's own design left soft.

## Philosophy

**Autonomy in the doing, a gate in the validating.** One mandatory human checkpoint — approving the spec — and everything from implementation through opening a draft PR runs unattended, inside an isolated git worktree, never touching your primary checkout.

**Trust nothing a faster stage claims.** QA re-runs the actual tests instead of trusting the implementer's word. Review re-checks QA's test-tampering and mutation-check claims independently instead of trusting QA's word. Each stage gets progressively more skeptical, not less.

**Guardrails are project data, not framework code.** What counts as a "critical path" in your project — the code that, if subtly wrong, costs you a bad week — lives in `.shipwright/guardrails.md` inside *your* project, not hardcoded into this plugin. You own and edit that file; the plugin only reads it.

**Risk-weighted, not one-size-fits-all.** A critical-path change gets fewer autonomous retries before stopping for you, not more — a second automated guess on a sensitive path is a worse failure mode than asking early. A trivial one-file, non-sensitive change can skip the approval gate entirely if you turn that on.

**No integration is load-bearing.** The default ticket "adapter" is a markdown file in your own repo. Nothing in the pipeline breaks if you never connect Linear, GitHub Issues, or anything else — those are optional adapters you can add later without touching the pipeline commands themselves.

## What's in here

```
commands/
  sw-init              one-time project setup (config.yml, guardrails.md)
  sw-ticket            turn a rough idea into a shaped ticket (local markdown by default)
  sw-spec              the interactive spec-gate — the one conversation stage
  sw-critical-impact   check files against the project's critical-path table
  sw-implement         spawn the implementer in an isolated worktree
  sw-qa                spawn read-only QA (tests, smoke recipe, mutation check, anti-tampering)
  sw-review            spawn the adversarial opus reviewer
  sw-pr-prep           preflight before opening a PR
  sw-pr-create         open the PR (always as a draft)
  sw-handoff           write the plain-language "how to test this" note
  sw-test-mutate       standalone mutation-check utility
  sw-deploy            optional, off by default, generic deploy stage
  sw-flow              the orchestrator that composes all of the above + status/resume

agents/
  sw-implementer       sonnet -> opus on critical-path/low-confidence escalation. Only one that edits code.
  sw-qa                sonnet, read-only.
  sw-reviewer           opus always, read-only, adversarial.

references/            templates + the state schema every command reads/writes against
  code-style.md          fixed baseline: naming, full SOLID, comments explain why-never-what
                         (default none), shallow nesting — ships with the plugin, NOT project-configurable
  file-organization.md   fixed baseline: one responsibility-type per file (interfaces, constants,
                         providers, services, controllers never coupled into one "God file") —
                         also fixed, and BLOCKING at review unlike code-style.md's NON-BLOCKING nits
```

## Quick start

```
# in a project you want this to manage:
claude plugin install shipwright   # or: install from this repo per Claude Code's plugin docs
/sw-init                           # writes .shipwright/config.yml and guardrails.md
/sw-ticket "add saved search to the client list"
/sw-flow add-saved-search          # spec -> [your approval] -> implement -> qa -> review -> draft PR -> handoff
```

Or skip the ticket step and just describe the work directly: `/sw-flow "add saved search to the client list"`.

Check on a run any time with `/sw-flow status`, or continue an interrupted one with `/sw-flow resume <ticket>`.

## Relationship to palazzo-engineering

This project generalizes an internal, team-scale system with the same shape (spec-gate, worktree-isolated implement/QA/review, draft-PR-only, never-weaken-a-test rules, critical-path escalation). Concretely, this version differs by:

| | palazzo-engineering | shipwright |
|---|---|---|
| Ticket system | Linear, load-bearing (ID scheme, directory names, entry modes all assume it) | Local markdown by default; any tracker is an optional, swappable adapter |
| State schema | No version field | `schema_version`, refuses to `resume` a mismatch instead of guessing |
| Rework budget | Flat cap (2) for every change | Risk-weighted: fewer autonomous retries on critical-path changes |
| Escalation trigger | Critical-path glob match only | Glob match **or** a stage self-reporting genuine low confidence |
| Critical-path table | Hand-maintained prose doc, team-wide | Per-project `.shipwright/guardrails.md`, seeded at init from the project's own docs |
| Smoke-testing | QA improvises per repo, ad hoc | Spec stage writes a declarative, mechanically-followable recipe QA just executes |
| Deploy | Separate, single-provider, hand-authored command with no link back to the ticket | Generic, optional, off by default, results recorded against the ticket's own trail |
| Mutation testing | Always the manual analytical check | Tries a real tool (Stryker/mutmut/etc.) first, falls back to the same manual check |
| Approval gate | Universal, non-negotiable, named human approvers | Same by default; optionally auto-approved for narrowly-defined trivial specs, since a solo project's only approver is you anyway |
| Code-style baseline | `CODE-STYLE.md`, one team-wide doc, SRP only | Ported and expanded as `references/code-style.md` — full SOLID (not just SRP), a harder line on comments ("code reads like a book," explicit forbidden-comment examples), same NON-BLOCKING-unless-it-causes-a-real-bug severity, shipped as a fixed part of the plugin so it's identical across every project |
| File/folder organization | Not addressed as its own concern | New: `references/file-organization.md` — one responsibility-type per file (interfaces, constants, providers, services, controllers never coupled into a "God file"). Fixed and **BLOCKING** at review by default, since this is precisely the debt an unsupervised implementer tends to accumulate |

## License

MIT — see [LICENSE](LICENSE).
