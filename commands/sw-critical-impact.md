---
description: Check a set of changed/to-be-changed files against the project's .shipwright/guardrails.md critical-path table. Reports HIT / WARN / MISS. Used internally by /sw-implement, /sw-spec, /sw-review, and /sw-pr-prep, and callable standalone.
argument-hint: Optional list of file paths (defaults to the current diff or working-tree changes)
---

# `/sw-critical-impact`

Binary first, heuristic second — reported in `references/guardrails-template.md`.

## Steps

1. **Resolve the file set.** From `$ARGUMENTS` if given; otherwise `git diff <base>...HEAD --name-only` if a branch is checked out, otherwise `git status --porcelain` for uncommitted changes.
2. **Read `.shipwright/guardrails.md`.** If it doesn't exist, say so plainly and report **MISS for everything** — do not invent a guardrail table from general knowledge of "what's usually sensitive." A missing guardrails file is a project-setup gap (point at `/sw-init`), not something to paper over with a guess.
3. **Match each file against the critical-path glob table.** Any match -> **HIT**, cite the matched glob and its "why" and "extra ritual" columns verbatim.
4. **For files with no glob match**, check the `keywords` list. A substring hit -> **WARN**, unless the file is also listed in `known_false_positives` (then MISS, and say why it's excluded).
5. **Report**: overall verdict for the set (HIT if any file HIT, else WARN if any WARN, else MISS), plus a per-file breakdown.

## Notes for callers

- **HIT** forces: the calling stage's model escalates to opus (if not already), and — if this is inside an orchestrated `/sw-flow` run — the extra ritual from `guardrails.md` is mandatory before `/sw-pr-create`, and the rework cap for this ticket drops to `config.rework_cap.critical_path`.
- **WARN** is informational — worth a human glance, does not force escalation or the extra ritual on its own.
- This command never edits `guardrails.md` — if it looks stale (a glob pointing at a file/symbol that no longer exists), say so and suggest the human update it, but don't silently "fix" someone's guardrail table out from under them.
