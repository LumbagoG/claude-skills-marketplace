# Clarifying Questions Guide

The clarification phase is where 80% of the value of this skill is created. Asking the wrong questions wastes the user's time. Asking the right ones means the rest of the plan writes itself.

## The four classes of questions every plan must cover

Every multi-area task can be derailed by ambiguity in one of these four dimensions. If you haven't asked at least one question in each class (or you have an evidence-backed answer from Phase 1), you're not ready to draft.

### 1. Modeling (where does the data live?)

The single most consequential decision. Is the new concept:
- A new top-level entity?
- A 1:1 extension of an existing entity?
- A few extra columns on an existing entity?
- A polymorphic association?

This decision propagates everywhere — schema, FSD slices, query patterns, indexing.

**Template:**
> "Как моделировать <концепт> в БД? У <existing entity> уже есть <hint>."
>
> Options (always 3, mark recommended first):
> 1. Отдельная модель X (1:1 с Y) (Recommended) — <when this wins, what it costs>
> 2. Расширить Y полями — <when this wins, what it costs>
> 3. Гибрид: модель X + флаг на Y как кэш — <when this wins, what it costs>

### 2. Cardinality (how many of what relate to what?)

m2m vs. 1:n vs. 1:1, with optional join-table state. This determines whether you need a junction model with its own status enum, or just an FK.

**Template:**
> "Как <X> привязывается к <Y>?"
>
> Options:
> 1. Many-to-many через XY с собственным статусом (Recommended for most cases) — гибко, требует junction model
> 2. One-to-many (X имеет primaryYId) — проще, но ограничивает рост
> 3. Независимы (нет связи) — обычно неверный выбор, но проверить

### 3. Integration depth (how deep is the new feature wired into existing flows?)

The "tip of the iceberg" question. The user usually says "I want X" but X usually depends on Y, and Y has invariants. This question forces them to declare those invariants.

Examples:
- New booking type → does it use existing `TableBooking` or create a parallel one?
- New email type → does it use existing notification preferences or new ones?
- New role → does it use existing auth callback or new one?

**Template:**
> "Как интегрировать <new flow> с существующим <old flow>?"
>
> Options:
> 1. Новая модель + опциональный FK на старую (Recommended) — чисто, lifecycle отдельный
> 2. Расширить старую модель полем — без новой сущности, но ломает инварианты в случае Z
> 3. Две независимые сущности без связи — просто, но нет каскадных операций

### 4. Lifecycle / state machine (what statuses, who can transition where?)

Implicit in any feature with workflow. If you skip this, you'll discover it during implementation — at high cost. Force the user to draw the state machine now.

**Template:**
> "Когда <actor> может <verb>? И что является терминальным состоянием?"
>
> Options for "complete a session":
> 1. Только tutor нажимает «complete» вручную (Recommended)
> 2. Auto-complete после endTime
> 3. Подтверждение обеих сторон

## Question hygiene

### Phrasing

- Each question ends with `?`
- Each option has a label (1–5 words) AND a description (1 sentence) AND when relevant a downside
- Mark the recommended option explicitly with `(Recommended)` suffix and put it first

### Volume

- 2–4 questions per round
- Max 2 rounds total before drafting
- If you reach Round 2, the questions should be **smaller** than Round 1 (refining trade-offs surfaced by Round 1 answers, not net new dimensions)

### After answers come back

Re-read every answer. Check for the following before drafting the plan:

1. **Internal consistency** — does choice A in Q1 contradict choice B in Q3? If yes, surface it before drafting:
   > "Один нюанс: ты выбрал «расширить TableBooking», но это ломает твой ответ на Q3 (опциональное бронирование без стола)."

2. **User accidentally chose default** — if the user picked the recommended option for everything, that's fine but suspicious. Briefly state in the draft what their choice implies, so they can object if it doesn't match what they had in mind.

3. **Open questions remaining** — if any answer was "Other" with free-form text, treat that as a deferred decision and put it in the artifact's "Open questions" section, not in the spec.

## Bad question examples

These are the questions junior planners ask. Don't.

❌ **"Public or private?"** — too generic. Frame as a concrete trade-off in context.

❌ **"What's the deadline?"** — irrelevant to architecture, save for later.

❌ **"Should we use TypeScript?"** — answer is in the project, you should know it.

❌ **"Do you want this to be testable?"** — yes, always. Don't ask.

❌ **"Should I use the existing pattern X or a new pattern?"** — usually existing. Don't ask, just use it and call it out as a decision.

❌ **"What naming convention?"** — read the codebase.

## Good question examples (from real plans)

✅ **"Trainer model: separate, extension on User, or hybrid?"** — the answer changes 5 files in opposite directions.

✅ **"When does the user 'I'll find a place myself' option produce a TableBooking record?"** — the answer determines whether `TableBooking.tableId` becomes nullable, which affects 50+ existing usages.

✅ **"Who sets training price: trainer once, or per-club, or per-session?"** — the answer determines whether you need a new `TrainerClub.hourlyRate` column.

✅ **"Cancellation: who can cancel, when, and does trust-index apply?"** — the answer determines whether the `cancellation-policy.ts` module is shared with bookings or separate.

## When to skip a class

You can skip an entire class only if Phase 1 (the silent project scan) gave you a confident answer. Examples:

- Project already has a `Trainer` model → skip "modeling" class.
- Project's auth-architecture page explicitly says "all entities scoped per club" → skip "cardinality" class for club-scoped entities.

In every other case, ask.

## Two-round pattern

The healthiest pattern looks like:

**Round 1** — modeling + cardinality + integration depth (3 questions in parallel)
→ user answers
→ you spot one inconsistency or one new dimension

**Round 2** — resolve inconsistency + cover lifecycle + cover any new dimension exposed by Round 1 (2–3 questions)
→ user answers
→ draft

Don't aim for one round if you'd compromise on coverage. Don't aim for three rounds if it's just because you're being indecisive.
