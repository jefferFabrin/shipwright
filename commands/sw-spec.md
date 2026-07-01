---
description: Interactive requirements + technical spec for a ticket. Walks the 5 canonical sections plus a technical approach, smoke test recipe, and impact survey. Writes spec.md with Status DRAFT. Stops at the human gate — never implements, never auto-approves except through the explicit trivial-spec path.
argument-hint: <ticket-id> or a freeform description for a brand-new one
---

# `/sw-spec`

The one stage designed to be a conversation, not a monologue. Entry A (no ticket yet — a freeform description) or Entry B (an existing ticket from `.shipwright/tickets/<id>.md` or whatever `ticket_adapter` is configured).

Input: $ARGUMENTS

## Steps

1. **Resolve entry mode.** If `$ARGUMENTS` matches an existing ticket id, load it (Entry B) — pull whatever context/notes it already has instead of re-asking from scratch. Otherwise treat it as a freeform description (Entry A) and create the ticket via `/sw-ticket`'s adapter call first.

2. **Read project context before asking anything**: `.shipwright/config.yml`, `.shipwright/guardrails.md`, the project's `CLAUDE.md`/README/existing specs if present. If the project has its own PRD or tech-spec docs (common right after `/sw-init` on a documented project), read the sections relevant to this ticket — don't make the human repeat what's already written down. Specs that come with their own "open questions / gaps" section (some do) are a gift: pull directly from it instead of re-deriving ambiguities from scratch.

3. **Walk the 5 canonical sections as a conversation**, asking concrete, specific questions rather than assuming — one round of batched questions, not twenty back-and-forths: Context, Current behavior, Expected behavior, Acceptance criteria, Out of scope. The Out-of-scope section is not optional filler — an empty one is a spec bug, not a sign there was nothing to exclude.

4. **Technical approach.** Propose a repo/base-branch and a 3-6 bullet approach; get a lightweight nod rather than a full design review here — implementation detail decisions belong to `/sw-implement`, not this gate.

5. **Smoke test recipe.** Write concrete, mechanically-followable verification steps (exact command or UI action -> exact expected result) that QA will later run verbatim. If you can't write concrete steps, that's a sign the Acceptance Criteria aren't concrete enough yet — go back and tighten them, don't hand QA a vague recipe.

6. **Impact survey.** Run `/sw-critical-impact`'s logic against the files this is likely to touch (best-effort at this stage, refined for real by the implementer later) and record HIT/WARN/MISS. Note if a cross-cutting/shared module is involved, whether a schema/migration is needed, and whether a third-party integration is touched.

7. **Trivial candidacy.** Set `Trivial: true` in the header **only if all of**: single file, impact survey says MISS, no schema/migration/dependency-manifest change. This is a claim, not a decision — `/sw-flow` still checks it against `config.auto_approve_trivial` before ever skipping the human gate.

8. **Write `spec.md`** (use `references/spec-template.md`) with `Status: DRAFT`.

9. **Stop.** Present the spec for approval. On approval: flip `Status: APPROVED`, and if Entry A, write the ticket via the adapter now (so a rejected/abandoned spec never litters the ticket tracker). On rejection or requested edits: revise and re-present — do not proceed with anything below GATE_A on a DRAFT spec, ever, standalone or orchestrated.

## What this command must never do

- Implement anything.
- Auto-approve a spec that doesn't meet every condition for `Trivial: true`, even if it looks obviously safe.
- Post to an external ticket tracker before the human has actually approved (Entry A) — a draft-then-rejected idea shouldn't leave a trace there.
- Skip the Out-of-scope section because "there's nothing to exclude" — push back and find something, there almost always is.
