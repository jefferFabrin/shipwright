# File & folder organization — shipwright fixed baseline

Companion to `references/code-style.md`. That file governs how a line/function/class is written; this one governs **where things live**. Same status as `code-style.md` — ships with the plugin, not project-owned, applies on every project. Unlike `code-style.md`'s line-level nits, a violation here is a real structural defect and is **BLOCKING** at review (see the severity note at the bottom) — this exists specifically because an unsupervised implementer's default habit is to dump everything into one file, and that habit is exactly what this guardrail exists to stop.

## The rule

**One responsibility-type per file.** Before writing to a file, name which single one of these it holds. The moment a second type wants in, extract it to its own file instead:

- **Interfaces/types** — type/interface/DTO definitions. Nothing else.
- **Constants** — literals, enums, magic numbers/strings, config defaults. Nothing else.
- **Providers** — infrastructure adapters: DB clients, external API/SDK wrappers, queue/cache clients, anything that talks to the outside world. This is the concrete side of the abstractions `code-style.md`'s Dependency Inversion rule asks business logic to depend on, not reach for directly.
- **Services** — business logic. Orchestrates providers and repositories to do the actual work. No route/HTTP/CLI concerns, no inline type definitions, no inline magic constants.
- **Controllers / handlers / routes** — the thin entry layer (HTTP route, CLI command, queue consumer). Parses input, calls a service, shapes the response. **No business logic lives here** — if there's an `if` deciding an outcome rather than just routing to it, that decision belongs in a service.
- **Repositories / data access** (where the project has this layer) — persistence queries only, no business rules about *what* the data means.

## The anti-pattern this exists to catch

```
# Avoid — one file, four responsibility-types coupled together
user.controller.ts
  export interface CreateUserRequest { ... }      // <- interface, wrong file
  const MAX_USERNAME_LENGTH = 32;                  // <- constant, wrong file
  export async function createUser(req) {
    if (req.username.length > MAX_USERNAME_LENGTH) // <- business rule, wrong file
      throw new Error(...);
    const db = new PostgresClient(...);            // <- concrete provider reached for directly
    return db.query(...);
  }
```

```
# Prefer — split by responsibility type
user.types.ts         export interface CreateUserRequest { ... }
user.constants.ts      export const MAX_USERNAME_LENGTH = 32;
user.service.ts        export async function createUser(req) { validates via the rule, calls the repository/provider }
user.controller.ts     export async function handleCreateUser(req, res) { parses req, calls userService.createUser, shapes res }
user.provider.ts (or providers/postgres-client.ts)   the actual DB client, injected into the service/repository, not instantiated inline
```

## How this adapts per project

The *principle* (separation by responsibility type, thin controllers, no God files) is fixed. The *exact folder names and layout* are not — follow whatever convention the project already has:

- If the project already has an established structure (a NestJS module's `dto/`/`entities/`/`services/`/`controllers/`, a Next.js App Router's route-handler + `lib/` split, a Django app's `models.py`/`views.py`/`serializers.py`, etc.), **match that convention** — don't impose the generic shape below over an existing, working one.
- If there's no established convention yet (a greenfield module, or a project just getting `/sw-init`'ed), default to a flat per-responsibility split alongside the file it's for: `<name>.types.ts`, `<name>.constants.ts`, `<name>.service.ts`, `<name>.controller.ts`, `<name>.provider.ts` — or the equivalent folders (`interfaces/`, `constants/`, `services/`, `controllers/`, `providers/`) once a module has enough files that flat-alongside gets crowded (roughly: more than 2–3 files of one type in a module is the signal to fold them into a subfolder).
- Record the convention you followed (or established) in `decisions.md` the first time a ticket touches a module with no prior convention — this becomes the reference point every later ticket in that module should match, instead of each ticket picking its own shape.

## What does *not* trigger this rule

- A small, genuinely single-purpose file that happens to define one tiny local type next to the one function that uses it (not exported, not reused elsewhere) is not a violation — this rule is about **coupling multiple responsibility-types together**, not about banning every inline type ever. The test is reuse/exposure: if another file would plausibly need to import that type/constant, it belongs in its own file now, not "whenever it comes up later."
- Don't split a 5-line module into 4 files to satisfy the letter of this rule — that's over-application in the other direction. The rule exists to stop coupling in modules big enough for it to matter, not to force ceremony on trivial code.

## Severity — how `sw-reviewer` applies this

This is review dimension 7, and unlike `code-style.md`'s line-level findings, it is **BLOCKING** by default: a new or substantially-rewritten file that visibly couples two or more responsibility-types (e.g. an inline interface *and* a business rule in a controller) is reported with a concrete split proposal (which pieces go to which new file), not just a note. It stays BLOCKING even though it isn't a "correctness" bug, because this is precisely the class of debt this guardrail is meant to prevent from accumulating unsupervised.

The one thing that downgrades a finding here to NON-BLOCKING: the coupling already existed before this ticket touched the file, and fully separating it is out of the ticket's scope (a broader refactor deserves its own ticket, not a scope-creeping "while I'm here" split, mirroring `code-style.md`'s "don't reformat untouched lines" rule and the implementer's own out-of-scope guard). Note this explicitly in the finding rather than silently letting pre-existing coupling slide.
