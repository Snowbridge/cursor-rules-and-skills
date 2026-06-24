---
name: spec-change-approval
description: Stops implementation when spec changes are required but not authorized in the task. Use when implementing from specifications/OpenAPI/contracts and a blocker would require editing specs, or when the user asks about changing specs during implementation.
---

# Spec change approval gate

## Core rule

If, while implementing code from a specification, you conclude that **changing the spec is required** to complete the task — and the task **does not explicitly authorize** those spec changes — **stop execution**. Do not edit specification files. Ask the user to approve a spec change or choose another approach.

## What counts as “changing the spec”

Any edit under specification sources of truth, including:

- `openapi.yaml`, `async.md`, `*-controller.md`, `readme.md` / `index.md` in service spec folders
- Messaging schema YAML under `messaging/`
- ADR or architecture docs **when used as contract**, if the task did not ask to update them

Codegen-only or build tooling changes are **not** spec changes unless they alter the published contract artifacts the user treats as spec.

## What counts as explicit authorization

The task clearly allows spec work, for example:

- “update the spec”, “fix openapi”, “sync spec with implementation”
- A plan/todo that lists spec file paths to create or change
- “spec and implementation” in scope together

Vague “make it work” or “implement the feature” **without** mentioning specs is **not** authorization.

## When to stop

Stop **before** editing spec files if any of these are true:

- OpenAPI/codegen cannot produce workable DTOs without restructuring schemas in the spec
- An endpoint, field, status code, or Kafka message in the spec appears wrong or incomplete for the stated behavior
- You would duplicate large schema blocks across multiple `openapi.yaml` files to work around tooling limits
- Implementation would diverge from the written contract (undocumented endpoints, different shapes, extra fields)

Continue **without** asking only when the fix stays entirely in `application/`, `libs/` (non-spec), migrations, tests, or tooling — and the public contract in the spec remains accurate and unchanged.

## What to do when stopped

1. **Do not** modify spec files or commit partial spec edits.
2. Summarize briefly:
   - what you were implementing
   - why the spec (or contract artifact) must change
   - minimal diff intent (files and nature of change, not full rewrites)
3. Offer **concrete options**, for example:
   - **A.** User approves spec change → you update spec, then codegen/implementation
   - **B.** User approves an alternative (codegen plugin, shared lib, Jackson workaround) with trade-offs
   - **C.** User clarifies task scope to include spec work
4. Wait for the user’s choice before proceeding.

## Approval request template

```markdown
## Spec change required (blocked)

**Task:** [one line]

**Blocker:** [why implementation cannot proceed without a spec or contract change]

**Proposed spec changes:** [files + short bullet list]

**Alternatives (no spec change):** [options with trade-offs, or “none viable”]

Please approve **A** (spec change), **B** (alternative: …), or adjust scope.
```

## Examples

### Stop — spec change not in task

Implementing `values-crud` from existing OpenAPI; generated `ValueDescriptor` polymorphism is broken. Duplicating dataype schemas into `values-crud/openapi.yaml` would change the spec. Task only says “implement values-crud”. → **Stop** and request approval.

### Continue — no spec change

Repository uses wrong jOOQ column; spec and OpenAPI are correct. Fix in `application/` only. → Continue.

### Continue — spec change authorized

Task: “Implement bundles-crud and update openapi if needed for validate-values batch limit.” → Spec edits allowed within that scope.

## Interaction with other skills

- If a platform skill forbids workarounds (e.g. raw `Map` instead of generated DTOs), do **not** bypass by silently changing the spec. Apply this gate instead.
- Finishing implementation after approved spec changes: run codegen/`buildAndPush` as required, then complete code and tests.
