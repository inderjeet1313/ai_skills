# Database Design — Design Principles (Technology-Agnostic)

A reusable set of design principles for modeling and evolving a persistence layer in **any** database
or stack. Technologies are named only as *examples* (`e.g.`) — none are required. The principles are
the contract; the engine (relational, document, key-value) and framework are interchangeable.

Each principle states **what** to do, **why** it matters, **how** to realize it neutrally, and **what
to watch for**.

---

## 0. The prime directive

> **The schema is the durable contract and the last line of defense for data integrity.**

Application code, caches, and clients come and go; the data outlives all of them. Invariants that
*must* always hold belong in the data layer where nothing can bypass them — not solely in application
code that a future service, script, or migration might sidestep. Model deliberately, constrain
strictly, and change carefully. Everything below serves this.

---

## 1. Model the domain; normalize by default, denormalize deliberately

- **What:** shape tables/collections around real domain entities and their relationships. Start
  normalized (each fact stored once); denormalize only as a measured, documented exception.
- **Why:** normalization removes update anomalies and contradictory copies of the same fact.
  Premature denormalization trades correctness for a performance win you may not need.
- **How:** one concept per table; relationships via keys; introduce redundancy only with a stated
  read-pattern justification and a plan to keep copies consistent.
- **Watch for:** the same fact editable in two places; a "wide" table accreting unrelated concerns;
  denormalization added "for speed" without a measurement.

## 2. Every row has a stable, meaningful primary key

- **What:** each entity has a single, immutable identifier.
- **Why:** identity that changes (or that encodes mutable business data) breaks references, caches, and
  audit trails.
- **How:** prefer a **surrogate key** (e.g. a generated UUID or sequence) decoupled from business
  attributes. Choose the generation strategy for your access pattern — sequential keys cluster and
  index tighter; random ids (e.g. UUID) avoid hotspots and hide volume/enumeration. Keep natural/business
  keys as **unique constraints**, not as the primary key.
- **Watch for:** composite keys made of mutable columns; business identifiers (email, account number)
  as the primary key; keys that leak sensitive info or sequential counts to clients.

## 3. Enforce integrity in the database, not only in code

- **What:** the invariants that must always hold are expressed as constraints the engine enforces.
- **Why:** application checks are bypassable (other services, bulk jobs, manual fixes, races). The
  database is the one gate every writer passes through.
- **How:** use `NOT NULL`, foreign keys, `UNIQUE`, and `CHECK`/domain constraints for real invariants;
  make columns non-nullable unless "unknown" is a genuine, meaningful state.
- **Watch for:** "we validate it in the app" as the sole guard; nullable columns used as a lazy default;
  orphaned rows because a relationship wasn't a foreign key.

## 4. Uniform metadata & audit columns via a shared base

- **What:** every persistent entity carries consistent housekeeping columns — creation time, last-update
  time, and a concurrency token — applied uniformly.
- **Why:** consistency enables generic tooling (auditing, debugging, sync, retention) and answers "when
  did this change?" everywhere the same way.
- **How:** a shared base definition (e.g. a mapped superclass / template) providing `created_at`,
  `updated_at`, and a `version` token, auto-populated on write. Update only changed columns where the
  engine supports it, to reduce write amplification and lock scope.
- **Watch for:** ad-hoc, per-table timestamp naming; timestamps set by clients instead of the system.

## 5. Control concurrency explicitly — never lose an update

- **What:** concurrent writers to the same row have a defined outcome; a read-modify-write never
  silently overwrites another writer's change.
- **Why:** the default of "last write wins" corrupts state under load — a classic lost-update bug.
- **How:**
  - **Optimistic** by default: a `version` column; the conflicting writer fails loudly and retries or
    is rejected. Cheap when contention is rare.
  - **Pessimistic** (row lock, "select for update") for genuinely hot rows: concurrent writers
    serialize on the row and each operates on a fresh snapshot — use for short critical sections.
  - Choose consciously per access pattern; make the choice visible in the code.
- **Watch for:** read-modify-write with no version token and no lock; long transactions holding locks;
  swallowing a concurrency exception instead of retrying or surfacing it.

## 6. Represent money and precise quantities exactly

- **What:** never store monetary or exactness-critical values in binary floating point.
- **Why:** floats can't represent most decimal fractions exactly — rounding drift accumulates into real
  financial errors.
- **How:** store as **integer minor units** (e.g. cents/fils) or an exact decimal type. Pick **one
  canonical unit convention**, document it, and convert only at display edges. Keep currency/unit
  alongside the amount.
- **Watch for:** `float`/`double` for money; mixed units across tables; amounts without an explicit
  currency/scale.

## 7. Store time as unambiguous instants

- **What:** timestamps are stored in a single, absolute reference (e.g. UTC instants / timezone-aware
  types), not naive local times.
- **Why:** local/naive times are ambiguous across zones and DST, making ordering and reconciliation
  unreliable.
- **How:** persist absolute instants; convert to local time only for presentation; store any relative
  or business date resolved to an absolute point.
- **Watch for:** "wall clock" timestamps without a zone; comparing times captured in different zones;
  storing relative dates ("in 30 days") instead of the resolved instant.

## 8. Evolve with versioned, forward-only, backward-compatible migrations

- **What:** all schema change flows through ordered, immutable, reviewed migration scripts applied
  automatically and identically in every environment.
- **Why:** reproducibility and auditability; "someone changed prod by hand" is how environments drift
  and data breaks.
- **How:**
  - Migrations are **ordered and append-only** — never edit a shipped migration; add a new one.
  - Prefer **expand/contract** for breaking changes: add the new shape, backfill, migrate readers/
    writers, then remove the old shape in a later release — so old and new code both run mid-deploy.
  - Keep migrations **backward-compatible within a deploy window** (rolling deploys run N and N+1
    simultaneously).
  - Make destructive steps deliberate, reviewed, and recoverable (backups/verification before drop).
- **Watch for:** manual production changes; editing a released migration; a single migration that both
  adds and drops such that the previous app version breaks the moment it runs.

## 9. Allow controlled flexibility — bounded, not a dumping ground

- **What:** sparse, evolving, or caller-defined attributes may live in a flexible column (e.g. a JSON/
  document field) instead of forcing a schema change per attribute.
- **Why:** some data legitimately varies or evolves fast; a rigid column-per-attribute causes migration
  churn. But flexibility trades away constraints, typing, and easy indexing.
- **How:** use flexible columns for genuinely open/optional data; keep anything you **filter, join,
  aggregate, or enforce** on as a real, typed, constrained column. Document the expected shape; version
  it; tolerate unknown fields on read for forward compatibility.
- **Watch for:** core queryable attributes buried in a JSON blob; the blob becoming an unversioned
  schema no one controls; validating nothing.

## 10. Index for real access patterns — measure, don't guess

- **What:** indexes exist to serve the queries you actually run; each one is justified by an access
  pattern.
- **Why:** missing indexes cause full scans; excess indexes slow every write and waste space. Both hurt.
- **How:** derive indexes from real query predicates, joins, sorts, and uniqueness needs; verify with
  the engine's query plan on representative data; add composite/covering indexes intentionally; remove
  unused ones.
- **Watch for:** indexing "just in case"; no index on a foreign key you join on; guessing instead of
  reading the plan; ignoring write-amplification cost of many indexes on a hot table.

## 11. Make effects idempotent and exactly-once

- **What:** retried or duplicated operations (client retries, at-least-once messaging) don't create
  duplicate rows or double effects.
- **Why:** networks and queues deliver duplicates; without a guard you get double charges/records.
- **How:** enforce dedup at the data layer with a **unique constraint** on a natural key or an explicit
  idempotency key; use insert-if-absent / upsert semantics; make the constraint the source of truth,
  not an application check-then-insert (which races).
- **Watch for:** check-then-insert without a unique constraint (TOCTOU race); idempotency enforced only
  in app memory/cache; no natural key to dedup on.

## 12. Delete deliberately — retention and auditability are design choices

- **What:** decide per entity whether deletion is physical (hard) or logical (soft), and how long data
  is retained.
- **Why:** regulated/financial and audited data often must be retained or provably removed; a blanket
  choice either loses history or violates minimization/erasure requirements.
- **How:** soft-delete (a flag/timestamp, excluded by default) where history/audit matters; hard-delete
  where minimization/erasure applies; define retention windows; keep an audit trail of significant
  changes independent of the live row.
- **Watch for:** soft-deleted rows silently included in queries/uniqueness; unbounded growth of "deleted"
  data; hard-deleting records you're required to retain.

## 13. Keep queries safe: bounded, paginated, and join-aware

- **What:** read paths return bounded result sets and don't explode into per-row follow-up queries.
- **Why:** unbounded reads and N+1 patterns are the most common causes of latency and outages under
  real data volume.
- **How:** paginate list endpoints (offset or keyset/seek); cap page size; fetch related data in a
  bounded number of queries; push filtering/sorting into the query, not the application; build dynamic
  filters compositionally rather than string-concatenating SQL.
- **Watch for:** "load all rows then filter in code"; a query per item in a loop; unbounded `IN`
  clauses; string-built queries (injection + plan churn).

## 14. Minimize and protect sensitive data; isolate tenants

- **What:** store the least sensitive data necessary; protect what you must keep; keep tenants'/users'
  data separable.
- **Why:** data you don't store can't leak; regulated data carries handling obligations; multi-tenant
  bleed is a breach.
- **How:** avoid persisting secrets/raw sensitive identifiers when a token/hash suffices; encrypt
  sensitive data at rest; apply least-privilege database access per service; enforce tenant scoping at
  the data layer (e.g. a tenant key on every row + a filter that can't be forgotten), not only in app
  logic.
- **Watch for:** secrets/PII stored "because it was easy"; one all-powerful DB account; tenant filtering
  left entirely to application code.

## 15. Make the schema reproducible and testable

- **What:** the schema and its evolution are verified automatically, and any environment can be rebuilt
  from scratch.
- **Why:** a schema you can't recreate or test is a schema you can't safely change.
- **How:** run migrations from empty in CI against a real, ephemeral instance of the target engine (not
  a mock/different engine); test constraints, uniqueness, and concurrency behavior; version seed/
  reference data; treat schema drift between environments as a defect.
- **Watch for:** testing against an in-memory engine that behaves differently from production; untested
  migrations; reference data edited by hand per environment.

---

## The model in one picture

```
        domain model
             │  normalize (denormalize only with a reason)
             ▼
   entities ─ stable surrogate keys ─ enforced constraints (NN / FK / UNIQUE / CHECK)
             │
   uniform metadata (created/updated/version)  +  explicit concurrency control
             │
   exact money & UTC instants   controlled-flexibility columns   intentional indexes
             │
   idempotent effects (unique keys)   deliberate deletion/retention   tenant isolation
             │
   evolve via ordered, backward-compatible migrations (expand → migrate → contract)
             │
   reproducible & tested from empty
```

## Design review checklist (any schema change, any engine)

- [ ] Real invariants enforced by DB constraints (NOT NULL / FK / UNIQUE / CHECK), not app-only.
- [ ] Stable surrogate primary key; business/natural keys are unique constraints.
- [ ] Uniform created/updated/version metadata present.
- [ ] Concurrent writes have a defined outcome (optimistic version and/or row lock) — no lost updates.
- [ ] Money/precise values are integer-minor-units or decimal, never float; one documented unit.
- [ ] Timestamps stored as absolute UTC instants.
- [ ] Change ships as a new, ordered, backward-compatible migration (expand/contract for breaking ones).
- [ ] Queryable attributes are typed columns; flexible/JSON used only for genuinely open data.
- [ ] Indexes justified by actual query plans; foreign keys you join on are indexed; no dead indexes.
- [ ] Duplicate/retry-safe via a unique/natural key at the DB, not a check-then-insert race.
- [ ] Deletion strategy (soft/hard) and retention are deliberate; soft-deletes excluded by default.
- [ ] Reads are bounded and paginated; no N+1; filters pushed into the query.
- [ ] Sensitive data minimized/encrypted; least-privilege access; tenant scoping enforced at the DB.
- [ ] Migration runs from empty in CI against the real engine; constraints and concurrency tested.
