---
description: Mutation-check a set of new tests — either by running a real mutation-testing tool if the project has one configured, or by proposing manual code mutations to verify the tests actually catch the bug they claim to (not fake assertions).
argument-hint: <path(s) to the new test file(s)>
---

# `/sw-test-mutate`

Input: $ARGUMENTS (test file paths, or a ticket id to derive them from `impl-notes.md`'s "Tests added" list)

## Steps

1. **Check `.shipwright/config.yml`'s `mutation_tool`.** If it names a real tool (Stryker, mutmut, cosmic-ray, PIT, etc.) and its config file is present in the repo, run it scoped to the files under test and report survived/killed mutants. A surviving mutant on a line the new test claims to cover is a FAIL for that test.

2. **If no real tool is configured** (`mutation_tool: auto` with nothing detected), do the manual analytical version:
   - Identify the specific code change the new test is supposed to guard.
   - On a scratch copy (never the actual worktree the human is about to commit from), revert just that change.
   - Re-run the new test. It **must fail**. If it still passes, the test isn't actually exercising the behavior it claims to — report which assertion is fake and why.
   - Restore the scratch copy afterward; this command never leaves the real worktree in a reverted state.

3. **Report**, per test: PASS (genuinely catches the mutation/revert) or FAIL (doesn't), with the specific reason for any FAIL.

This is human-in-the-loop by design when no real tool exists — it's a targeted proposal-and-check, not a full mutation-testing suite reimplemented from scratch. If a project does enough of this to be worth it, the right move is installing a real mutation-testing tool for its primary language and pointing `config.yml`'s `mutation_tool` at it, not asking this command to get cleverer.
