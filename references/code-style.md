# Code style standards — shipwright fixed baseline

Readability standards for any code `sw-implementer` writes or substantially rewrites. Unlike `.shipwright/guardrails.md`, this file is **not** project-owned — it ships with the plugin and applies the same way on every project, because it's about maintainability, not project-specific risk. It's about maintainability, not correctness, so `sw-reviewer` treats it as **NON-BLOCKING** by default (see the severity note at the bottom).

These apply to *new or substantially-rewritten* code only. Don't reformat untouched lines to satisfy them — that's a mixed-scope change (the implementer's own "touch only what's in scope" rule already forbids this). When the surrounding file has a strong local convention that conflicts with one of these, match the file and flag the divergence rather than rewriting the world.

## 1. Names reveal intent

Variable, function, and class names say *what they are / what they do*. A reader should understand a name without chasing its definition.

- **No single letters, no cryptic abbreviations** — not `x`, `d`, `tmp`, `val`, `data`, `res`, `arr`. (Conventional loop counters `i`/`j` in a tight numeric loop are the one tolerated exception.)
- **Name by the thing, not the type** — `customer` over `obj`, `excludedRoles` over `arr2`.
- **Functions are verbs that name the work** — a function that fetches a customer's data is `fetchCustomerData()`, not `getData()`. A boolean reads as a predicate — `isExpired`, `hasPermission`.
- **Acronyms only when they're the domain term** — `url`, `id`, `api`, `sftp` are fine (they *are* the vocabulary). Invented short forms are not.

| Instead of | Write |
|---|---|
| `get_data()` | `fetch_customer_data()` |
| `const r = await q(...)` | `const results = await queryOrders(...)` |
| `function proc(x)` | `function normalizeAccountId(rawId)` |
| `let flag` | `let isCriticalPathHit` |

## 2. One function, one responsibility (SRP)

A function does **one thing** at one level of abstraction. If you can't summarize it without "and", it's probably two functions.

- Extract a named helper when a block has its own clear purpose (validation, mapping, the actual I/O). The extraction's payoff is the *name* — it documents intent better than a comment would.
- Size is a smell, not a hard limit — there's no line cap. A 60-line function that does one linear thing can be fine; a 15-line function juggling parsing + a network call + formatting is not. Watch for mixed levels of abstraction in one body.
- A function that both *decides* and *does* (computes a value **and** persists it) is two responsibilities — split them so each is testable in isolation. Logic that isn't isolated enough to unit-test on its own is exactly where a subtle bug hides behind a green test suite.

## 3. Comments explain *why*, not *what* — and default to none

Names carry the *what* (rules 1–2). A comment earns its place only for a non-obvious *why*:
- a critical-path invariant or gotcha a future reader would otherwise break (see the project's own `.shipwright/guardrails.md`),
- a deliberate technical-debt/workaround marker (`TODO`/`FIXME` with the reason and, ideally, a ticket id),
- a genuinely non-obvious *why* (an unintuitive ordering, a vendor/spec quirk, a deliberate deviation from the obvious approach).

If you need a comment to explain *what* a line does, rename until you don't. Match the surrounding file's comment density when in doubt — fewer, not more. Doc comments/JSDoc on a public API surface are not "inline comments" and are fine where the codebase already uses them.

## 4. Shallow nesting — early return

Handle errors and base cases at the top and return; keep the happy path at the lowest indentation. Deep `if/else` pyramids hide the main flow and are where edge-case bugs live.

```ts
// Avoid — happy path buried, easy to misread the merge logic
function resolveDiscount(order) {
  if (order) {
    if (order.customer) {
      if (isEligible(order.customer)) {
        return computeDiscount(order);
      }
    }
  }
  return 0;
}

// Prefer — guards first, happy path flat
function resolveDiscount(order) {
  if (!order?.customer) return 0;
  if (!isEligible(order.customer)) return 0;
  return computeDiscount(order);
}
```

- Guard clauses for invalid/empty/error inputs come first.
- Avoid `else` after a `return` — the `else` branch becomes the unindented continuation.
- If you're past ~3 levels of nesting, extract or invert.

## Severity — how `sw-reviewer` applies these

These are **NON-BLOCKING** findings under review dimension 6 (style/maintainability): flagged as suggestions with `file:line`, but they never trigger REWORK on their own and never gate a PR by themselves. A correct, well-tested change ships even with a naming nit.

The exception is when a style problem *causes* a correctness or critical-path risk — nesting so deep a merge-logic bug hides in it, or a name so misleading a caller wires the wrong field. At that point it's already caught by the **Correctness** or **Critical-path** review dimension and inherits that severity. Don't reclassify a real bug as a style nit to soften it, and don't reclassify a pure style nit as BLOCKING to force it.
