# `.shipwright/guardrails.md` — critical-path table (project-specific)

This file is seeded once per project (by `/sw-init` or by hand) and maintained by whoever owns that project — Shipwright itself never edits it. `/sw-critical-impact`, `sw-implementer`, and `sw-reviewer` all read it at runtime; none of them hardcode paths.

Detection is **binary first, heuristic second**:
1. A file path matches a glob in the table below -> **HIT**. Not a judgment call.
2. No glob match, but the path contains one of the free-text keywords in "Heuristic keywords" -> **WARN** (lower severity — a human should glance at it, but it doesn't force the Opus escalation or the extra ritual).
3. Neither -> **MISS**.

## Critical-path table

| Glob | Why it's critical | Extra ritual on HIT |
|---|---|---|
| _(seed per project — see example below)_ | | |

**Extra ritual, applied whenever any file in a change matches HIT** (mirrors what a serious production incident usually would have caught):
1. Local build + lint + full affected test suite + a manual click-through of the golden-path flow, with evidence (command output / screenshot) recorded in `impl-notes.md`.
2. Reviewer must be Opus (already the default for `/sw-review`, called out here so it's never silently downgraded).
3. `review.md` must include an explicit rollback plan: confirm `git revert` is sufficient, or name the extra steps (migration down, cache bust, feature-flag kill switch).
4. `pr-body.md` must state a rollout/watch plan: what to look at after merge and for how long, even for a solo project (a log line, an error-rate check, a manual re-test).

## Heuristic keywords (WARN layer)

List free-text substrings here that hint at sensitivity without a hard path match yet (e.g. a new module you haven't formalized into the glob table). Keep a short "known false positive" list next to it so the WARN doesn't cry wolf on files you've already checked and cleared.

```
keywords: []
known_false_positives: []
```

## Recommended starting seed — CRM Reuters / Vitra Imobi

Derived from SPEC-Backend and SPEC-Frontend's own explicit acceptance criteria and gap sections. Copy into the table above once `apps/`/`packages/` exist and adjust the globs to the real paths that land.

| Glob | Why it's critical | Extra ritual on HIT |
|---|---|---|
| `packages/db/**/rls*`, `packages/db/**/policies/**` | Row-Level Security is the tenant-isolation boundary. Spec calls this "the most important test in the entire project" — a leak here means account A reads account B's data. | Run the RLS test suite against a second seeded tenant, not just the default one; confirm no query path bypasses RLS via a service-role key. |
| `packages/entitlements/**` | Single gate for plan/limit/AI-credit checks. Spec explicitly flags scattered `if (plan === ...)` as the anti-pattern to prevent. | Grep the diff for any new plan/limit conditional living outside this package. |
| `packages/ai-service/**`, any router calling it | AI credit debit must happen **before** calling the paid provider — named acceptance criterion. Debiting after risks spending money with no credit backing it. | Confirm order of operations in the diff: credit check+debit precedes the provider call in every code path, including error/retry branches. |
| `packages/api/**/routers/billing/**`, Stripe webhook handlers | Billing correctness, proration, downgrade blocking (`canDowngradeTo`). | Test against Stripe's webhook test fixtures / CLI, not just happy path; confirm idempotency on webhook replay. |
| `packages/api/**/routers/agency/**deactivateBroker**` | Must hard-fail without `reassignToUserId` or it orphans client/deal data — named acceptance criterion, client- and server-side. | Confirm both a UI-level test and an API-level test exist for the missing-reassignment case. |
| `apps/web/**/api/public/**`, any unauthenticated route | No-login public surface (`leads`, `landing-events`) — bot/spam exposure with no auth wall. | Confirm rate limiting + bot mitigation (e.g. Turnstile) is present and not accidentally disabled. |
| Anything writing `lead_captures.consent_lgpd` or geolocation consent | Two distinct LGPD consent flows the spec says must not be conflated; both need client- and server-side enforcement. | Confirm the exact mandated consent text and that the server rejects a request missing consent, not just the client UI. |
| `packages/db/**/audit_log*`, commission-rule / client-delete / plan-change / broker role-change paths | Spec mandates before/after audit snapshots for these specific mutations. | Confirm an audit_log row is written in the same transaction as the mutation, not best-effort after. |

## Heuristic keywords for this project

```
keywords: ["rls", "policy", "entitlement", "credit", "stripe", "webhook", "consent", "lgpd", "audit"]
known_false_positives: []
```
