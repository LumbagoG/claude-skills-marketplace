# Example: Trainers Integration

This is a real run of the skill from start to finish — read it once when learning the skill to calibrate the level of detail expected at each phase.

The user's request was: "интегрировать функционал тренеров" (integrate trainers feature) into a table tennis platform with:
- public catalog of trainers
- per-club trainer section
- personal trainer page
- trainer application form → admin queue → email
- reviews from students
- training booking with optional table booking
- email + socket notifications to trainer
- profile section for managing one's trainings

Project context: elo-tt — Next.js 16 + React 19 + Prisma 7 + PostgreSQL + Socket.io + NextAuth.js 5, FSD architecture, Russian-first.

---

## Phase 1 — Discover (silent project scan)

Files read in order:

1. `wiki/_meta/hot.md` — got stack, current focus (Forum Phase 12J), HeroUI v3 GOTCHA list, glass-style rule.
2. `wiki/_meta/index.md` — saw existing entities/features/widgets to understand naming conventions.
3. `apps/web/prisma/schema.prisma` — skimmed `User`, `ClubApplication`, `ClubStaffApplication`, `TableBooking`, `ClubTable`, `ClubSchedule`. Noticed `User.isTrainer Boolean @default(false)` already exists.
4. `wiki/database/db-club-application.md` + `wiki/features/feature-club-applications.md` — application flow already exists for clubs; reuse pattern.
5. `wiki/business-logic/biz-table-booking.md` + `wiki/features/feature-table-booking.md` — understood booking model.
6. `wiki/architecture/auth-architecture.md` + `wiki/architecture/realtime-architecture.md` — knew about Socket rooms and role system.
7. `wiki/tasks/_template.md` — confirmed artifact format.

Total: ~6500 tokens. Did NOT read: forum modules (irrelevant), tournament module (irrelevant), entire codebase.

After Phase 1, the questions to ask wrote themselves.

---

## Phase 2 — Clarify (Round 1)

Four questions, each with 3 options, recommended option first.

1. **Modeling** — separate Trainer model (1:1 User) vs. extend User vs. hybrid. Recommended: separate. User picked: separate.

2. **Cardinality** — Trainer↔Club: m2m via TrainerClub vs. 1:n vs. independent. Recommended: m2m. User picked: m2m.

3. **Integration depth** — booking integration: TrainingSession with optional `tableBookingId` vs. extend `TableBooking` with `trainerId` vs. parallel models. User picked: extend `TableBooking`. **This was a mistake** — see Phase 2 Round 2 below.

4. **Lifecycle (review)** — when can a student review: only after completed session vs. anytime vs. multiple per session-pair. User picked: only after completed, **and clarified one review per student-trainer pair, not per session**. Plus: "тренер должен иметь свою админку для управления тренировками — там же он ставит что тренировка завершена".

That last clarification was the most valuable single sentence — it surfaced the entire trainer-dashboard requirement, which became Phase 6 of the plan.

---

## Phase 2 — Clarify (Round 2)

The answer to Q3 contradicted the user's original prompt ("опционально, должны дать возможность пользователю выбрать клуб либо пользователь сам решит где провести тренировку"). Surfaced this:

> "Один важный нюанс: ты выбрал «расширить TableBooking полем trainerId», но в исходном промпте сам отметил, что бронирование стола — опционально. Если тренировка живёт внутри TableBooking, то модель ломается, когда стола нет (поле tableId — NOT NULL)."

Asked four refining questions:

1. **Booking-table integration (revisit)** — three options. User picked the recommended: separate TrainingSession with optional tableBookingId, plus a comment field for "I'll figure it out".

2. **Pricing** — trainer self-set rate vs. per-club vs. per-session. User picked: trainer self-set (single hourlyRate) + flag "table rent included".

3. **Trainer dashboard scope** — multi-select. User picked all four: incoming requests, calendar, complete-button, profile editing — and added: **"Также на главной странице в админке должен быть подсчет заработка с возможностью фильтрации по клубам, датам, периодам (сегодня, вчера, неделя, месяц, год)"**.

4. **Cancellation** — three options. User picked: trust-index for both sides + club rules respected + trust-index visible in trainer's request queue.

Two rounds gave full coverage. Drafted plan after Round 2.

---

## Phase 3 — Design

### 3.1 Schema (verbatim from final plan)

5 new models + 4 enums + extension to `User`:

```prisma
enum TrainerStatus { ACTIVE PAUSED ARCHIVED }
enum TrainerApplicationStatus { PENDING ACCEPTED REJECTED }
enum TrainerClubStatus { PENDING_CLUB_APPROVAL ACTIVE REJECTED REMOVED }
enum TrainingSessionStatus { PENDING CONFIRMED REJECTED CANCELLED COMPLETED NO_SHOW }

model Trainer {
  id String @id @default(cuid())
  userId String @unique
  user User @relation("UserTrainerProfile", fields: [userId], references: [id], onDelete: Cascade)
  bio String @db.Text
  specialization String[] @default([])
  yearsOfExperience Int @default(0)
  certificates String[] @default([])
  preview_img String?
  hourlyRate Int           // в копейках
  includesTableRent Boolean @default(false)
  status TrainerStatus @default(ACTIVE)
  approvedAt DateTime
  approvedById String?
  approvedBy User? @relation("TrainerApprovedBy", fields: [approvedById], references: [id], onDelete: SetNull)
  averageRating Float @default(0)
  reviewsCount Int @default(0)
  completedSessionsCount Int @default(0)
  applications TrainerApplication[] @relation("TrainerOriginatingApplication")
  clubs TrainerClub[]
  sessions TrainingSession[]
  reviews TrainerReview[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([status, averageRating])
  @@index([userId])
}

// + TrainerApplication, TrainerClub, TrainingSession, TrainerReview
```

Key index choices, with stated reason:
- `Trainer @@index([status, averageRating])` — каталог фильтрует ACTIVE и сортирует по рейтингу.
- `TrainingSession @@index([trainerId, status, completedAt])` — earnings aggregations group by completed sessions per trainer per period.
- `TrainerReview @@unique([trainerId, authorId])` — enforces "1 review per pair" at DB level.

### 3.2 API contracts (verbatim)

```typescript
// POST /api/trainer-applications  (любой авторизованный USER)
type SubmitTrainerApplicationInput = {
  fullName: string;
  phone: string;
  city: string;
  bio: string;
  yearsOfExperience: number;
  specialization: string[];
  certificateUrls: string[];
  hourlyRate: number; // в копейках
  preferredClubIds: string[];
  includesTableRent: boolean;
  motivationLetter: string;
};

// PATCH /api/trainer-applications/[id]  (ADMIN/SUPERADMIN)
type ReviewTrainerApplicationInput =
  | { action: "approve" }
  | { action: "reject"; reason: string };

// POST /api/training-sessions  (USER)
type CreateTrainingSessionInput = {
  trainerId: string;
  startTime: string;
  endTime: string;
  clubBooking?: { clubId: string; tableId: string };
  customLocation?: { comment: string };
};

// PATCH /api/training-sessions/[id]  (TRAINER own / STUDENT own)
type UpdateTrainingSessionInput =
  | { action: "confirm" }
  | { action: "reject"; reason: string }
  | { action: "cancel"; reason?: string }
  | { action: "complete" }
  | { action: "no_show" };
```

### 3.3 Layer table — full FSD mapping (~30 rows in real plan)

Showed every entity, feature, widget, view, route, API, socket file, middleware change, i18n, tests. ~30 rows. Implementer has complete coverage of "every directory I'll touch".

### 3.4 Architectural decisions (8 entries, each one-line + reason)

Examples:
- **Trainer как 1:1 с User, а не подмодель** — User уже перегружен (player+club+staff fields). Trainer — отдельный концепт со своим жизненным циклом, ценой, отзывами.
- **TrainerClub.PENDING_CLUB_APPROVAL зеркалит ClubStaffApplication** — переиспользует знакомый клубам UX-паттерн.
- **`TableBooking.trainerId`? — НЕТ.** Связь живёт обратно — `TrainingSession.tableBookingId` (опциональная FK с `@unique`). Это сохраняет инварианты `TableBooking` и позволяет тренировке существовать без стола.
- **`trainerPriceSnapshot` на TrainingSession** — защита от изменения `Trainer.hourlyRate` после подачи заявки. Цена фиксируется в момент создания.

### 3.5 Risks (12 entries)

Examples:
- ⚠️ Транзакция «booking + session» — должны быть в одной prisma transaction. Иначе race с другим пользователем который параллельно бронирует.
- ⚠️ Cascade-cancel TableBooking — при cancel TrainingSession с linked tableBooking → cascade `tableBooking.status = CANCELLED`, иначе стол «висит».
- ⚠️ TrainerClub vs ClubStaffApplication — это РАЗНЫЕ модели. Не реюзать ClubStaffApplication для тренеров.
- ⚠️ User.isTrainer cache desync — единственное место где этот флаг должен меняться: транзакции в trainer-application-service.ts. Покрыть unit-тестом.
- ⚠️ Один отзыв на пару — `@@unique([trainerId, authorId])` + проверка anchor-условия (sesion.status = COMPLETED).
- ⚠️ HeroUI v3 broken tokens — для всех новых badge/chip применять `bg-zinc-*/N` через `tv()`. Glass-style для карточек обязателен.

---

## Phase 4 — Phases (13 phases total)

| # | Phase | What | Owner |
|--|--|--|--|
| 0 | Pre-flight | UX + storybook аудит для всех новых UI | ux-senior + storybook-expert (parallel) |
| 1 | Schema + entities | 5 моделей + 4 enum + миграция + entity-слой | senior-backend-dev |
| 2 | Application flow | Форма заявки + admin queue + 3 email | backend + frontend |
| 3 | Public catalog & profile | view trainers + view trainer-public-profile | frontend |
| 4 | Club integration | widget club-trainers + tab in clubs/* | frontend + backend |
| 5 | Booking flow | wizard + transactional booking + email + socket | backend + frontend |
| 6 | Trainer dashboard | requests + calendar + profile + earnings | backend + frontend |
| 7 | User profile section | Tab «Тренировки» в /profile | frontend |
| 8 | Reviews | StarRating + form + anchor validation | backend + frontend |
| 9 | Cancellation + trust-index | Подключить cancellation-policy ко всем cancel handlers | backend |
| 10 | Realtime + i18n | Socket room + events + ru/en keys | frontend |
| 11 | E2E happy path | Playwright spec | senior-qa-engineer |
| 12 | QA gate | pnpm validate + screenshots | senior-qa-engineer |

Each phase produces an independently reviewable commit.

---

## Phase 5 — Document

Created `wiki/tasks/2026-04-29-001-trainers-integration.md` from `wiki/tasks/_template.md` with all sections filled. Updated `wiki/tasks/index.md` (added row to "Очередь").

Total artifact length: ~600 lines including verbatim user prompt, all clarifications, schema, API, layer table, decisions, risks, phases, open questions, related wiki pages.

---

## Lessons from this run

1. **Not asking the right Round-2 question would have been a costly bug.** The user picked "extend TableBooking with trainerId" in Round 1 even though it contradicted their own prompt. A planner that just drafts on Round 1 answers would produce an unimplementable plan. The clarify phase exists exactly to catch this.

2. **The dashboard requirement was hidden.** It came from a single phrase in the answer to a different question. Always re-read the user's free-text answer for buried requirements.

3. **Existing patterns saved time.** Application flow → reused `ClubApplication` shape. Email templates → reused `ApplicationAcceptedEmail` skeleton. Cancellation → reused `trust-index.ts`. The plan ended up with much less code than a from-scratch design would have produced.

4. **Open questions are non-trivial.** The artifact's "Open questions" section ended up with 7 entries (chat, payments, recurring sessions, group sessions, FTS search, etc.) — all real future work, all explicitly deferred. A plan without an open-questions section is hiding scope.

5. **The artifact is the deliverable, not the chat.** The chat summary was 100 words; the artifact was 600 lines. The 100 words is what the user reads first; the 600 lines is what survives in the wiki and what the implementer reads when they sit down to code.
