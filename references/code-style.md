# Code style standards — shipwright fixed baseline

Readability standards for any code `sw-implementer` writes or substantially rewrites. Unlike `.shipwright/guardrails.md`, this file is **not** project-owned — it ships with the plugin and applies the same way on every project. It covers naming, SOLID, comments, and nesting at the line/function/class grain; `references/file-organization.md` covers the file/folder grain (where things live, not how they're written) and is judged with different, stricter severity — read both together.

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

## 2. SOLID

Not decoration — each letter is a concrete, checkable rule applied at whatever grain the language supports (class, module, function, file).

**S — Single Responsibility.** A function/class/module does **one thing** at one level of abstraction, and has exactly one reason to change. If you can't summarize it without "and", it's probably two.
- Extract a named helper when a block has its own clear purpose (validation, mapping, the actual I/O). The extraction's payoff is the *name* — it documents intent better than a comment would.
- Size is a smell, not a hard limit — there's no line cap. A 60-line function that does one linear thing can be fine; a 15-line function juggling parsing + a network call + formatting is not.
- A function that both *decides* and *does* (computes a value **and** persists it) is two responsibilities — split them so each is testable in isolation. Logic that isn't isolated enough to unit-test on its own is exactly where a subtle bug hides behind a green test suite.

**O — Open/Closed.** Adding a new case should mean *adding* code, not *editing* a growing conditional that already handles N other cases.
- A `switch`/`if-else` chain that gains a new branch every time a new type/variant appears is a signal to introduce a small abstraction (strategy map, polymorphic handler, registry) instead of the Nth branch.
- Don't over-apply this to something that will only ever have 2–3 cases forever — that's premature abstraction, the opposite failure mode. Reach for O/C when you can already see the branch count is going to keep growing.

**L — Liskov Substitution.** Anything implementing an interface/base type must be swappable for another implementation without breaking the caller's expectations.
- An implementation that throws on inputs the interface's contract says are valid, or that silently no-ops where the contract implies an effect, violates this even if it "type-checks."
- If an implementation can't honestly satisfy the interface, the interface is wrong for that case — don't force it in with a special-cased caller check instead (`if (impl instanceof SpecialCase) { ... }` at every call site is the tell).

**I — Interface Segregation.** Don't force something to depend on methods/fields it doesn't use.
- A fat interface with 12 methods where each implementer only really needs 3 should split into smaller, client-specific interfaces.
- If implementing an interface means writing 5 stub/no-op/`throw new Error("not implemented")` methods, the interface is doing too much.

**D — Dependency Inversion.** Business logic depends on abstractions (interfaces/ports), not concrete implementations; concrete details are injected, not reached for directly.
- A service importing and instantiating a concrete external client (a specific DB driver, a specific HTTP SDK) directly, instead of depending on an injected interface the project already has a provider/adapter pattern for, is the violation to catch — especially once a project has established that pattern anywhere (see `references/file-organization.md`'s `providers/` convention).
- Don't invent a DI framework for a script that will only ever have one implementation — this rule bites when a second implementation (a mock, a test double, an alternate provider) is realistically coming and the code makes swapping it painful.

## 3. Code reads like a book — comments explain *why*, never *what*, and default to none

If a reader has to stop and puzzle out *what* a block does, that's a naming/structure failure, not a missing-comment problem — fix the name or extract a function, don't paper over it with a comment. The code itself, read top to bottom with well-named functions as its "paragraph headings," should be the documentation.

A comment earns its place **only** for a non-obvious *why* that no amount of renaming could carry:
- a critical-path invariant or gotcha a future reader would otherwise break (see the project's own `.shipwright/guardrails.md`),
- a deliberate technical-debt/workaround marker (`TODO`/`FIXME` with the reason and, ideally, a ticket id),
- a genuinely non-obvious *why* (an unintuitive ordering, a vendor/spec quirk, a deliberate deviation from the obvious approach).

Delete on sight any comment that just narrates the line under it — these are explicitly forbidden, not a matter of taste:
```ts
// loop through the users        <- delete: the loop already says this
for (const user of users) { ... }

// set isActive to true          <- delete: the assignment already says this
user.isActive = true;

// call the payment service      <- delete: the function name already says this
await paymentService.charge(order);
```
If you catch yourself about to write a comment and it starts with "loop", "set", "call", "check if", "get the" — stop, that's a *what*-comment, delete the instinct instead of the comment after the fact. Match the surrounding file's comment density when in doubt — fewer, not more. Doc comments/JSDoc on a public API surface are not "inline comments" and are fine where the codebase already uses them.

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
