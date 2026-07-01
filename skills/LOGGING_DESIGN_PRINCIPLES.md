# Logging Module — Design Principles (Technology-Agnostic)

A reusable set of design principles for building a centralized logging / audit / observability module
in **any** language or stack. Technologies are named only as *examples* (`e.g.`) — none are required.
The principles are the contract; the tools are interchangeable.

Each principle states **what** to do, **why** it matters, **how** to realize it neutrally, and **what
to watch for**.

---

## 0. The prime directive

> **Application code emits neutral, structured events. It must not format, serialize, or route them.**

Emitting a fact ("payment X settled, amount Y") is the caller's job. Deciding how it *looks* (text vs
JSON), *where* it goes (console, file, collector), and *what is stripped* (PII/secrets) are downstream
concerns owned by the module. This single separation is what makes logging swappable, testable, and
safe. Everything below serves it.

---

## 1. Separate emission from formatting and routing

- **What:** callers produce `event + key/value pairs`; a downstream encoder decides the wire format and
  a configuration layer decides the destination.
- **Why:** you can switch text↔JSON, add a sink, or change a field's shape without touching a single
  call site. Formatting in call sites means every format change is a code migration.
- **How:** provide an emit API that takes structured data; render format in a pluggable
  encoder/appender configured outside code.
- **Watch for:** string concatenation building a "log message," or callers choosing an output target.
  Both are leaks of a downstream concern upward.

## 2. Structured, machine-first events

- **What:** a log entry is a set of typed key/value fields, not a sentence. A human-readable `message`
  is one field among many.
- **Why:** logs are queried, aggregated, and alerted on by machines. Prose is unsearchable and brittle.
- **How:** first-class key/value fields; a stable field schema; consistent field names.
- **Watch for:** interpolating values into the message string (`"user " + id + " failed"`) instead of
  attaching `userId=id` as a field.

## 3. A thin, uniform facade over the logging backend

- **What:** all code logs through one small facade, never the underlying logger directly.
- **Why:** one place to enforce structure, guards, and redaction; the backend (log library) becomes an
  implementation detail you can replace.
- **How:** an `emit(level, event)` facade wrapping the platform logger; immutable event builder;
  discourage/forbid direct backend or `stdout` use.
- **Watch for:** `print`/`console`/raw-logger calls scattered in business code — they bypass every
  guarantee below.

## 4. Logging must never change business behavior

- **What:** a logging failure is invisible to the business path. Logging is best-effort and side-effect
  free with respect to the operation being logged.
- **Why:** an observability concern must never cause an outage or alter a financial/critical result.
- **How:** wrap all capture logic in try/catch and continue; null-safe emit is a no-op; never block on
  logging I/O in the request path (buffer/async at the sink); only *genuine business* exceptions
  propagate through instrumentation.
- **Watch for:** an exception in a redactor/serializer/enricher escaping into the caller; synchronous
  network logging on the hot path.

## 5. Think of it as a pipeline with clean layers

```
emit  →  enrich/capture  →  redact  →  correlate  →  route  →  export
```

- **What:** each stage has one responsibility and is independently replaceable/testable.
  - *emit* — produce the event (facade).
  - *capture* — gather cross-cutting context (request, operation, caller identity).
  - *redact* — remove/mask sensitive data.
  - *correlate* — attach ids that tie related events together.
  - *route* — send to the right stream/channel.
  - *export* — ship to storage/analysis.
- **Why:** mixing stages (e.g. redacting inside a caller, routing from code) destroys the ability to
  reason about or change any one concern.
- **Watch for:** any stage reaching across into another's job.

## 6. Security by default: mandatory, fail-closed, policy-driven redaction

- **What:** sensitive data (PII, secrets/credentials, tokens, financial identifiers) is scrubbed before
  anything is written — as a rule, not an option.
- **Why:** a secret or personal identifier written to a log is a breach that outlives the log line
  (backups, indexes, downstream copies).
- **How:**
  - **Fail-closed:** when a secret is detected, drop/replace the *whole* payload rather than risk a
    partial leak. Prefer over-redaction to under-redaction.
  - **Tiered:** apply the right technique per context — coarse redaction for free-form payloads;
    field-level masking for structured diffs; **one-way hashing/fingerprinting** for values you must
    *correlate* but never reveal.
  - **Policy-driven:** detection rules (patterns/field lists) live in configuration, not hardcoded, so
    they evolve without code changes.
  - **Context-aware allowances:** any exception (e.g. an internal channel permitted more detail) is an
    explicit, narrow policy — deny by default.
  - **Observe detection, not the value:** record *that* sensitive data was present (a boolean/flag)
    without recording the data.
  - **Bound size:** truncate captured payloads to a max length; never log unbounded input.
- **Watch for:** best-effort/partial masking of secrets; redaction logic duplicated per call site;
  patterns baked into code.

## 7. Correlate and propagate context

- **What:** related events across a request, an operation, and process boundaries share correlation ids
  (e.g. a request id and a trace/span id).
- **Why:** the value of logs is in reconstructing a *flow*, not reading isolated lines. Without shared
  ids, distributed behavior is unreadable.
- **How:** hold correlation ids in an ambient context store for the current unit of work (e.g.
  thread-local / scoped / async-context); establish or inherit an id at every entry point; **propagate
  ids across asynchronous or network boundaries** by carrying them in message/request metadata and
  restoring them on the far side; always clear the context when the unit of work ends.
- **Watch for:** context leaking between units of work (not cleared), or ids dropped at async hand-offs
  (queue, thread pool, outbound call) — the trace breaks exactly where it's most needed.

## 8. Declarative, non-invasive instrumentation

- **What:** cross-cutting capture (audit, timing, tracing) is attached declaratively (annotation/
  decorator/middleware/interceptor), not hand-written inside business methods.
- **Why:** business code stays focused; instrumentation is consistent and centrally evolvable; you can
  enable/disable it without editing logic.
- **How:** mark an operation as "audited/traced" declaratively; a single generic interceptor reads the
  marker, captures inputs/changes/outcome, and emits — business code contains no logging statements.
- **Watch for:** copy-pasted logging boilerplate in every handler; instrumentation entangled with
  business branches.

## 9. Configurable, optional, and gracefully degrading

- **What:** every capability is independently switchable, overridable, and safe to omit.
- **Why:** different environments and consumers need different behavior; a logging module must not
  force a dependency or crash when an optional integration is absent.
- **How:** a single, namespaced configuration surface; per-capability on/off flags with sensible
  defaults; treat integrations (tracing exporter, message-bus hooks, HTTP capture) as **optional** —
  detect capability at runtime and no-op cleanly when absent; let the host override any component.
- **Watch for:** always-on behavior that can't be disabled; hard dependencies on an optional
  telemetry/transport library that break builds or startup when it's not present.

## 10. Be cheap on the hot path

- **What:** logging runs on every request/operation/message, so its overhead must be near-zero when not
  needed.
- **Why:** observability that degrades throughput gets turned off — and then you're blind.
- **How:** guard construction behind level checks; compile patterns and cache reflective/metadata
  lookups once; **true no-op when a feature is disabled** (check the flag before allocating anything);
  bound captured payload sizes; prefer immutable, allocation-light structures.
- **Watch for:** building expensive strings/objects before checking whether they'll be logged;
  recompiling patterns per call; allocating context for a disabled feature.

## 11. Consistency and no magic strings

- **What:** field names, event names, and context keys come from a single registry and follow a stable
  schema.
- **Why:** downstream queries/dashboards/alerts depend on exact, stable field names; drift silently
  breaks them.
- **How:** centralize key/field/event-name constants; document the schema; review changes to it.
- **Watch for:** the same concept logged under different keys in different places.

## 12. Design for evolution

- **What:** the log/audit schema will change; changes must not break existing consumers.
- **Why:** dashboards, alerts, and compliance pipelines are built on today's shape.
- **How:** additive changes by default; for breaking changes use a **dual-emit / migration window**
  (emit both old and new during transition, migrate consumers, then retire the old); provide extension
  points (new markers, new patterns, new sinks) so extension doesn't require editing the core.
- **Watch for:** renaming/removing a field in place; forcing consumers to migrate atomically.

## 13. Testability is a first-class requirement

- **What:** the module is verified by tests, with **redaction as the highest-priority suite**.
- **Why:** you cannot manually eyeball every log line for leaks; correctness of scrubbing is a security
  control, not a nicety.
- **How:** table-driven tests for every PII/secret pattern and the fail-closed path; assert emitted
  fields and levels via a capturing sink; assert context is set and cleared; test the disabled/no-op
  paths and graceful degradation when an optional integration is missing.
- **Watch for:** shipping a new capturable field without a test proving it can't carry sensitive data.

---

## The model in one picture

```
        ┌────────── application code ──────────┐
        │  emit structured event (facade only) │
        └───────────────┬──────────────────────┘
                        ▼
   enrich/capture  →  redact  →  correlate  →  route  →  export
   (context)         (mandatory,  (shared ids,  (streams/  (storage/
                      fail-closed) propagated)   channels)  analysis)
        every stage: independently configured, optional, fail-safe, tested
```

## Design review checklist (any logging change, any stack)

- [ ] Emits structured key/values through the facade — no prose messages, no raw backend/`stdout`.
- [ ] No sensitive data can reach a sink; secrets fail-closed; correlate-only values are hashed.
- [ ] Redaction rules are configuration, not code; applied in one place, not per call site.
- [ ] A logging failure cannot break or alter the business operation.
- [ ] Correlation ids are established at entry, propagated across async/network boundaries, and cleared.
- [ ] Cross-cutting capture is declarative; business code has no logging boilerplate.
- [ ] Every capability is flag-disable-able, host-overridable, and no-ops cleanly when an optional
      integration is absent.
- [ ] Hot path is cheap: level-guarded, cached, bounded, true no-op when disabled.
- [ ] Field/event/context names come from a central registry; schema is stable or migrated additively.
- [ ] Tests cover redaction (first), emitted fields, context lifecycle, and degraded modes.
