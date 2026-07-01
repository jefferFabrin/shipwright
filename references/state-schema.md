# state.json schema

Machine state behind `flow.md`, written and owned only by `/sw-flow`. Stage commands invoked standalone (outside an orchestrated run) never write this file.

```jsonc
{
  "schema_version": 1,               // bump on any breaking field change; /sw-flow refuses to `resume` a state.json with a schema_version it doesn't recognize and stops for a human instead of guessing
  "ticket": "add-saved-search",      // slug, not tied to any external tracker
  "title": "Add saved search filters to the client list",
  "entry": "A",                      // "A" = greenfield description, "B" = existing local ticket file
  "stage": "IMPLEMENT",              // SPEC | GATE_A | IMPLEMENT | QA | REVIEW | PR_DRAFT | HANDOFF | DONE | BLOCKED
  "gates": {
    "spec_approved": false,
    "auto_approved": false           // true if auto-approved by the trivial-spec heuristic (config.auto_approve_trivial) instead of a human click-through
  },
  "risk": {
    "critical_path": "MISS",         // HIT | WARN | MISS, from /sw-critical-impact
    "matched_globs": []
  },
  "model_per_stage": {               // what actually ran, post-escalation, per stage
    "spec": "sonnet",
    "implement": "sonnet",
    "qa": "sonnet",
    "review": "opus"
  },
  "escalations": [
    // { "stage": "implement", "trigger": "critical_path_hit" | "self_reported_low_confidence", "reason": "...", "from_model": "sonnet", "to_model": "opus", "at": "2026-06-30T18:04:00Z" }
  ],
  "rework": {
    "count": 0,
    "cap": 2,                        // resolved from config.rework_cap.critical_path or .default depending on risk.critical_path
    "history": []                    // [{ "cycle": 1, "from_stage": "review", "reason": "...", "at": "..." }]
  },
  "repo": ".",                       // path relative to the project root; single-repo by default (see references/multi-repo.md for the optional multi-repo variant)
  "worktree": "../.shipwright-worktrees/add-saved-search",
  "branch": "sw/add-saved-search",
  "links": {
    "ticket_file": ".shipwright/tickets/add-saved-search.md",
    "pr": null,
    "handoff_file": null
  },
  "timestamps": {
    "spec": "2026-06-30T17:40:00Z",
    "gate_a": "2026-06-30T17:55:00Z",
    "implement": "2026-06-30T18:00:00Z",
    "qa": null,
    "review": null,
    "pr_draft": null,
    "handoff": null
  },
  "blocked_reason": null             // always set when stage == BLOCKED, e.g. "rework cap hit", "spec defect — needs re-spec", "unresolvable base branch"
}
```

## Rules

- `stage` only moves forward, with one exception: `REVIEW -> IMPLEMENT` for a rework cycle.
- `BLOCKED` is reachable from any stage and always carries a `blocked_reason`. Nothing auto-recovers from `BLOCKED` — a human runs `/sw-flow resume <ticket>` after fixing the underlying issue (re-approving a spec, resolving a base branch, deciding on a rework past the cap).
- `schema_version` must be checked before `resume`. If it doesn't match the version this plugin currently writes, stop and tell the human what changed rather than silently reinterpreting old fields.
- Any field not in this schema (a one-off bolt-on for a special run, e.g. a `"mode": "qa-rework"` marker) is allowed but must be namespaced under an `"extensions": {}` object, never a bare top-level key — keeps future schema diffs mechanical.
