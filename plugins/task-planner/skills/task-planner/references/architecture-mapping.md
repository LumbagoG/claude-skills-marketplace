# Architecture Mapping Guide

Phase 3.3 of the workflow asks for a table that lists every layer/file the work touches. The exact column set depends on the project's architecture. This file gives you templates per architecture style.

The goal is the same regardless of architecture: **the implementer should be able to scan one table and know every directory they'll edit**.

---

## Feature-Sliced Design (FSD)

The strictest layered architecture in the JS world. Layers are: `app → views → widgets → features → entities → shared`. Imports go DOWN only.

Use this column set:

| Layer | File / path | Action |
|--|--|--|
| db | `prisma/schema.prisma` | + N models, + M enum |
| db | `prisma/migrations/<NNN>_<name>/` | new migration |
| entity | `src/entities/<name>/` | create (model + api + ui) |
| feature | `src/features/<name>/` | create |
| widget | `src/widgets/<name>/` | create |
| view | `src/views/<name>/` | create |
| route | `app/[locale]/<path>/page.tsx` | create / edit |
| api | `app/api/<path>/route.ts` | create |
| socket | `apps/socket-server/src/<file>.ts` | edit (allowlist + rooms) |
| middleware | `src/middleware.ts` | edit (auth gates) |
| messages | `messages/{ru,en}.json` | edit (i18n keys) |
| tests | `src/**/__tests__/*.test.ts` | create |
| e2e | `apps/e2e/e2e/<name>.spec.ts` | create |

### FSD-specific rules to surface in the plan

- **Cross-layer imports only via public API** (`index.ts` of the target slice) — call this out for each new slice.
- **Slice naming** — match existing conventions. If the project has `feature-club-applications`, your new feature should be `feature-trainer-applications`, not `feature-trainerApplications`.
- **`server.ts` pattern** — if the project separates `index.ts` (client-safe) from `server.ts` (Prisma queries), apply it to every new entity.
- **Cross-feature composition** — if two features need to render together but FSD forbids cross-feature imports, use the `extraSlot: ReactNode` injection pattern at the widget/view level.

---

## Hexagonal / Ports-and-Adapters

Layers: `domain → application → adapters → infrastructure`. Domain has no IO.

| Layer | File / path | Action |
|--|--|--|
| domain | `src/domain/<aggregate>/` | + entity, + value objects |
| domain | `src/domain/<aggregate>/events/` | + domain events |
| application | `src/application/<aggregate>/use-cases/` | + use case classes |
| application | `src/application/<aggregate>/ports/` | + repository interface |
| adapters in | `src/adapters/in/http/` | + HTTP controller |
| adapters in | `src/adapters/in/cli/` | + CLI command (if applicable) |
| adapters out | `src/adapters/out/persistence/` | + repository impl + ORM mapping |
| adapters out | `src/adapters/out/messaging/` | + event publisher impl |
| infrastructure | `src/infrastructure/<service>.ts` | edit (DI wiring) |
| db | migrations | + schema migration |
| tests | `tests/unit/domain/`, `tests/integration/` | create |

### Hexagonal-specific rules

- **Domain has no IO** — surface explicitly that the new domain entity must not import any adapter.
- **Use cases are stateless** — wire them through DI, don't `new` them inside controllers.
- **Repository interface lives with the use case**, not with the persistence adapter.

---

## Clean Architecture

Layers: `entities → use cases → interface adapters → frameworks`.

| Layer | File / path | Action |
|--|--|--|
| entities | `src/core/entities/` | + entity classes |
| use cases | `src/core/use-cases/<feature>/` | + use case + DTOs |
| interface | `src/interfaces/controllers/` | + controller |
| interface | `src/interfaces/presenters/` | + presenter / view model |
| interface | `src/interfaces/gateways/` | + gateway interface |
| frameworks | `src/frameworks/web/routes/` | + route handler |
| frameworks | `src/frameworks/db/repositories/` | + repository impl |
| frameworks | `src/frameworks/email/` | + email adapter |

---

## MVC / "Plain Rails-style"

Layers: `models → controllers → views`. Often plus services and policies.

| Layer | File / path | Action |
|--|--|--|
| model | `app/models/<name>.rb` | + model |
| migration | `db/migrate/<ts>_<name>.rb` | + migration |
| controller | `app/controllers/<name>_controller.rb` | + controller |
| view | `app/views/<name>/` | + templates |
| service | `app/services/<name>_service.rb` | + service object |
| policy | `app/policies/<name>_policy.rb` | + authorization |
| mailer | `app/mailers/<name>_mailer.rb` | + mailer |
| job | `app/jobs/<name>_job.rb` | + background job |
| route | `config/routes.rb` | edit |
| test | `spec/` or `test/` | + specs |

---

## Plain Next.js (App Router, no FSD)

Layers: `app/` (routing) + `components/` (UI) + `lib/` (logic) + `prisma/` (data).

| Layer | File / path | Action |
|--|--|--|
| db | `prisma/schema.prisma` | + models |
| route | `app/<path>/page.tsx` | create |
| route | `app/<path>/layout.tsx` | edit (if needed) |
| api | `app/api/<path>/route.ts` | create |
| component | `components/<feature>/<Name>.tsx` | create |
| lib | `lib/<feature>/<verb>.ts` | + server actions / utils |
| email | `emails/<Name>.tsx` | + react-email template |
| middleware | `middleware.ts` | edit |
| tests | `__tests__/` or co-located | + tests |

---

## Custom architectures

If the project's architecture is neither of the above:

1. Skim 3–5 directories at the project root.
2. Identify the **layering convention** — even if it's informal, there's usually one.
3. Build a custom column set that matches.

Generic fallback (works for almost anything):

| Concern | File / path | Action |
|--|--|--|
| Data | <wherever schema lives> | + new model |
| Business logic | <wherever rules live> | + module |
| HTTP | <wherever endpoints live> | + endpoint |
| UI | <wherever components live> | + components |
| Side effects | <wherever email/queue/etc lives> | + new effects |
| Tests | <wherever tests live> | + tests |

---

## Tips that apply to every architecture

- **Group by intent, not by file count.** "Add 8 i18n keys" is one row, not 8.
- **Distinguish create vs. edit.** Edit is riskier (regression surface). Surface them visually.
- **Show migration order.** If a non-obvious order matters (ADD nullable → backfill → ALTER NOT NULL), put it in the architecture decisions, not buried in the table.
- **List middleware/guards explicitly.** Auth changes are a frequent forgotten requirement.
- **List i18n explicitly** if the project is internationalized. The total number of new keys is a useful sanity-check on UI scope.

## What NOT to put in this table

- "Run lint" — that's a QA-gate phase, not a layer.
- "Add tests" without saying which tests — tests live in the same row as the code being tested, or in a dedicated tests row with a path.
- Vague entries like "various components" — name them.
