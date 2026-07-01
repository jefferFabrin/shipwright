# `.shipwright/<ticket>/impl-notes.md` template

Written by `sw-implementer`.

```markdown
# Impl notes: <ticket>

## Branch & base
- Base branch: <branch> (resolved how: <e.g. "config default" | "confirmed with human">)
- Working branch: sw/<ticket>
- Worktree: <path>
- Verification: `git -C <worktree> status` / `git -C <worktree> log --oneline -1`

## Risk
- Critical-path: HIT / WARN / MISS — matched globs: [...]
- Escalation: none | escalated to opus, reason: <critical_path_hit | self_reported_low_confidence> — <detail>

## Test baseline (captured before any code change, on the clean base)
- Command(s) run: `<...>`
- Result: <pass count / fail count / last lines of output>

## Files touched (in-scope only)
- `<path>` — <one-line why, tied to an AC>

## Commits
- `<sha>` — <message> (no AI co-author trailer)

## Local validation evidence
- Build: `<command>` -> <result>
- Lint: `<command>` -> <result>
- Tests (post-change): `<command>` -> <result, compared to baseline>

## Tests added / changed
- Added: `<path>` — covers AC #<n>
- Changed (pre-existing test): `<path>` — justification: see decisions.md entry <date> — assertion is stricter-or-equal, not weaker
```

## Rules this feeds

- `sw-qa` treats a missing or vague "Test baseline" section as a FAIL — the baseline must exist and be from the *clean* base, not post-change.
- Any "Changed (pre-existing test)" entry without a matching `decisions.md` justification is BLOCKING at both QA and review — independently re-checked at review, not trusted from QA's word.
