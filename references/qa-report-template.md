# `.shipwright/<ticket>/qa-report.md` template

Written by `sw-qa`. Read-only stage — never fixes anything it finds.

```markdown
# QA report: <ticket>

## Gates
- Build: PASS / FAIL — `<command>`
- Type-check (if separate from build): PASS / FAIL — `<command>`
- Lint: PASS / FAIL — `<command>`
- Existing tests: PASS / FAIL — `<command>` (a runner that fails to even start is a FAIL, never an assumed pass)

## Acceptance criteria
| # | AC | Observed result | Evidence | Verdict |
|---|---|---|---|---|
| 1 | <from spec.md> | <what actually happened> | <command output / screenshot path / curl response> | PASS/FAIL |

## Smoke test recipe execution
Run every step from spec.md's "Smoke test recipe" verbatim and record what happened — this is mechanical, not a judgment call.
| Step | Expected | Observed | Verdict |
|---|---|---|---|

## Anti-tampering scan
`git diff <base>...HEAD` restricted to test files.
| File | Hunk type (weakened / removed / skipped / xfail / strengthened / new) | Justified in decisions.md? | Verdict |
|---|---|---|---|
Any weakened/removed/skipped/xfail hunk without a matching, AC-tied `decisions.md` justification is BLOCKING — full stop, route to rework, regardless of how the rest of QA looks.

## Mutation check (on newly added tests only)
- Tool used: <stryker | mutmut | manual-analytical>
- If manual: revert the implementation's fix, re-run the new test -> must FAIL. If it still passes, the test is fake and this whole QA stage is FAIL.
- Result: <surviving mutants / manual revert result>

## Affected-flow evidence
Concrete evidence the golden path actually works end to end beyond the AC table — request/response pair, before/after screenshots, log line, whatever fits the change.

## Result
PASS -> proceed to /sw-review
FAIL -> route back to /sw-implement with the specific failing items listed above (do not restate the whole report, just what must change)
```
