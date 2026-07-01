---
description: Guided dialog to produce a local ticket (or, via a configured adapter, an external one) in the canonical shape. Writes the ticket file; does not implement anything.
argument-hint: Optional short description of the work
---

# `/sw-ticket`

Turns a rough idea into a well-shaped ticket, using the `ticket_adapter` configured in `.shipwright/config.yml` (default: `local`, writing to `.shipwright/tickets/<id>.md`).

Initial description, if given: $ARGUMENTS

## Steps

1. **Resolve an id.** Slugify the description (kebab-case, 2-5 words). If a ticket with that slug already exists, ask whether this is the same piece of work (append to it) or a new one (add a numeric suffix).
2. **Ask only what's missing** — don't interrogate if the description already answers something:
   - What's the problem or opportunity? (context)
   - What happens today, concretely?
   - What should happen instead, concretely?
   - Any acceptance criteria already in mind?
   - Anything explicitly out of scope for this pass?
3. **Write the ticket** via the adapter's `write(id, { title, body, status: "open" })`. For `local`, that's `.shipwright/tickets/<id>.md`:
   ```markdown
   ---
   title: <title>
   status: open
   ---

   ## Context
   ...
   ## Notes
   (anything gathered above that isn't yet a formal spec — /sw-spec turns this into one)
   ```
4. **Report the id and file path** and mention `/sw-flow <id>` (Entry B) is how this turns into a spec + implementation, or `/sw-spec <id>` to just do the spec stage standalone.

This command never writes `spec.md`, never implements, and never opens a PR — it only produces the ticket that a later `/sw-spec` will turn into an approved spec.
