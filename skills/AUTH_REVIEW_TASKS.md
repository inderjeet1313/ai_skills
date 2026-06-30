# Auth Module — Remediation Tasks

> Source: code review of the `auth` module on branch `feature/auth-module` (2026-06-30).
> Audience: AI coding agents. Each task is self-contained — context, exact file references,
> implementation guidance, and acceptance criteria. Work tasks in priority order
> (CRITICAL → HIGH → MEDIUM → LOW) unless told otherwise.

## How to use this file

- Pick the highest-priority unchecked task.
- Read the referenced files **before** editing. Do not assume; verify line numbers, they drift.
- Keep changes scoped to the task. Do not refactor unrelated code.
- Match existing code style (tabs for indentation, constructor injection, records for DTOs/domain).
- Add/adjust tests for every behavioral change. Run `./gradlew test` before marking done.
- Tick the checkbox and add a one-line note when complete.

## Module map (for orientation)

| Concern | File |
|---|---|
| Token issue/login/refresh/logout/me | `auth/src/main/java/com/einvoicing/auth/service/AuthService.java` |
| JWT create/parse/sign | `auth/src/main/java/com/einvoicing/auth/security/JwtTokenService.java` |
| Refresh token issue/validate/revoke | `auth/src/main/java/com/einvoicing/auth/service/RefreshTokenService.java` |
| Refresh token persistence | `auth/src/main/java/com/einvoicing/auth/repository/RefreshTokenRepository.java` |
| Auth config / properties | `auth/src/main/java/com/einvoicing/auth/config/AuthProperties.java`, `AuthSecurityConfig.java` |
| Request filters | `auth/src/main/java/com/einvoicing/auth/security/JwtAuthenticationFilter.java`, `TenantContextFilter.java` |
| Controllers | `tenant-api/.../TenantAuthController.java`, `admin-api/.../AdminAuthController.java` |
| App config | `app/src/main/resources/application.yaml`, `application-local.yml` |
| Migrations | `app/src/main/resources/db/migration/` |

---

## CRITICAL

### [ ] C1 — Prevent the hardcoded JWT secret from reaching production

**Problem.** `AuthProperties.java` defaults `jwtSecret` to the public string
`"change-me-local-development-secret-change-before-production"`, and
`application.yaml` uses the same fallback. There is no startup guard. If
`APP_AUTH_JWT_SECRET` is unset in production, the app boots with a publicly known
secret and **anyone can forge admin JWTs** (`area=ADMIN`, arbitrary
`platformRole`) — a complete authentication bypass.

**Files.**
- `auth/src/main/java/com/einvoicing/auth/config/AuthProperties.java`
- `app/src/main/resources/application.yaml`
- `app/src/main/resources/application-local.yml`

**Implementation.**
1. Remove the hardcoded `jwtSecret` default from `AuthProperties` (keep `issuer`/TTL defaults).
2. Remove the hardcoded fallback in `application.yaml` (`${APP_AUTH_JWT_SECRET}` with no default).
3. Keep a dev secret **only** in `application-local.yml`.
4. Add a fail-fast validator that runs at startup for non-`local` profiles. Reject when:
   - secret is null/blank, or
   - secret starts with `change-me`, or
   - secret is shorter than 32 bytes (HS256 requires a ≥256-bit key).

   Implement via a `@PostConstruct`-annotated validator bean, an
   `ApplicationRunner`, or a `@ConfigurationProperties` validator. Throw
   `IllegalStateException` with a clear message so the container crashes loudly.

**Acceptance criteria.**
- App fails to start in a non-`local` profile when the secret is missing, default, or weak.
- App starts normally in `local`.
- A test verifies the validator rejects the default/short secret and accepts a strong one.

---

## HIGH

### [ ] H1 — Add brute-force protection to login & refresh

**Problem.** `AuthService.login` has no rate limiting or lockout. `/api/auth/login`
and `/api/admin/auth/login` are open to credential stuffing / brute force.

**Files.**
- `auth/src/main/java/com/einvoicing/auth/service/AuthService.java`
- `tenant-api/.../TenantAuthController.java`, `admin-api/.../AdminAuthController.java`
- Possibly a new `auth/.../security/LoginRateLimiter.java`

**Implementation.**
- Add per-account **and** per-IP throttling on login (and refresh). Options, simplest first:
  - In-memory bucket (e.g. `bucket4j`, or a `ConcurrentHashMap` of attempt counters with a sliding window) — acceptable for single instance.
  - Persistent/distributed counter (DB or Redis) if multi-instance is a near-term goal — confirm deployment topology before choosing.
- On exceeding the threshold, return HTTP 429 with a generic message. Do not leak whether the account exists.
- Reset the counter on successful authentication.
- Make limits configurable via `app.auth.*` properties.

**Acceptance criteria.**
- After N failed attempts within the window, further attempts return 429 until the window passes.
- Successful login resets the counter.
- Tests cover threshold trip, window reset, and success-resets-counter.
- **Decision needed from a human:** single-instance vs distributed. If unclear, implement the in-memory version behind an interface so it can be swapped.

### [ ] H2 — Eliminate user enumeration via login timing

**Problem.** In `AuthService.login` (`findByEmail(...).orElseThrow(...)` then
`passwordEncoder.matches(...)`), the no-such-user path skips the bcrypt
comparison and returns faster than the wrong-password path, allowing email
enumeration by timing.

**Files.** `auth/src/main/java/com/einvoicing/auth/service/AuthService.java`

**Implementation.**
- When the user is not found, still perform a bcrypt comparison against a fixed
  dummy hash (a precomputed bcrypt of a random string) before throwing, so both
  paths take comparable time.
- Keep the response message identical for both cases ("Invalid email or password").

**Acceptance criteria.**
- Unknown-email and wrong-password paths both invoke the password encoder.
- Response and status code are identical for both.
- A test asserts `passwordEncoder.matches` is invoked even when the user does not exist (e.g. via a mock/spy).

### [ ] H3 — Decide & document access-token revocation on logout

**Problem.** `logout` revokes only the refresh token. The access token (15m TTL)
stays valid until expiry; there is no `jti`/denylist.

**Files.**
- `auth/src/main/java/com/einvoicing/auth/service/AuthService.java`
- `auth/src/main/java/com/einvoicing/auth/security/JwtTokenService.java` (if adding `jti`)

**Implementation (choose one, get human sign-off).**
- **Option A (accept the trade-off):** document that access tokens are valid until
  expiry after logout; keep TTL short. Lowest effort. Add a code comment + note in `AGENTS.md`.
- **Option B (denylist):** add a `jti` claim and a short-lived denylist (in-memory
  or Redis) checked in `JwtAuthenticationFilter`. Revoke `jti` on logout. Higher
  effort; only worth it if immediate admin revocation is a requirement.

**Acceptance criteria.**
- A deliberate decision is recorded (comment + `AGENTS.md`).
- If Option B: logout makes the access token unusable on the next request; test proves it.

---

## MEDIUM

### [ ] M1 — Detect refresh-token reuse (stolen-token response)

**Problem.** `refresh` rotates correctly (revoke old, issue new), but replay of an
already-revoked token simply returns "invalid" with no reaction. A leaked token
that was already used should trigger a family-wide revocation.

**Files.**
- `auth/src/main/java/com/einvoicing/auth/service/RefreshTokenService.java`
- `auth/src/main/java/com/einvoicing/auth/repository/RefreshTokenRepository.java`
- `auth/src/main/java/com/einvoicing/auth/service/AuthService.java`

**Implementation.**
- When a token is found but is **revoked** (not merely absent), treat it as reuse:
  revoke all active refresh tokens for that `user_id` and reject.
- Add `RefreshTokenRepository.revokeAllForUser(UUID userId, Instant now)`.
- Distinguish "not found" from "found-but-revoked" in `RefreshTokenService.findValid`
  (or add a dedicated lookup) so `AuthService.refresh` can react.
- Optionally log a security warning.

**Acceptance criteria.**
- Replaying a revoked token revokes the user's remaining tokens and returns 401.
- A genuinely unknown token still returns 401 without family revocation.
- Tests cover both paths.

### [ ] M2 — Purge expired/revoked refresh tokens

**Problem.** `refresh_tokens` rows accumulate forever; nothing deletes expired or
revoked entries.

**Files.**
- `auth/src/main/java/com/einvoicing/auth/repository/RefreshTokenRepository.java`
- A new scheduled component, e.g. `auth/.../service/RefreshTokenCleanupJob.java`
- `app/.../AppApplication.java` (ensure `@EnableScheduling` is present, or add a `@Configuration` for it)

**Implementation.**
- Add `RefreshTokenRepository.deleteExpiredAndRevoked(Instant now)` →
  `DELETE FROM refresh_tokens WHERE expires_at < ? OR revoked_at IS NOT NULL`.
- Add a `@Scheduled` job (e.g. daily) that calls it. Make the cron configurable.
- Confirm scheduling is enabled app-wide without breaking tests (guard with profile if needed).

**Acceptance criteria.**
- Expired and revoked rows are deleted by the job.
- A repository test verifies the delete predicate.
- Scheduling does not interfere with the test suite.

### [ ] M3 — Lock down Swagger / api-docs outside `local`

**Problem.** `AuthSecurityConfig` `permitAll`s `/v3/api-docs/**`, `/swagger-ui.html`,
`/swagger-ui/**` in every profile, and `application.yaml` enables springdoc globally.

**Files.**
- `auth/src/main/java/com/einvoicing/auth/config/AuthSecurityConfig.java`
- `app/src/main/resources/application.yaml`, `application-local.yml`

**Implementation.**
- Disable springdoc/swagger-ui by default; enable only in `local` (move the
  `springdoc.*` enable flags to `application-local.yml`, set `enabled: false` in base).
- Keep the `permitAll` matchers but ensure the endpoints are absent/disabled in prod,
  or gate the matchers behind a profile.

**Acceptance criteria.**
- Swagger UI and api-docs are reachable in `local` and not served in prod profile.

### [ ] M4 — Make `me()` read-only transactional

**Problem.** `AuthService.me()` performs DB reads but isn't transactional, unlike
`login`/`refresh`.

**Files.** `auth/src/main/java/com/einvoicing/auth/service/AuthService.java`

**Implementation.** Annotate `me()` with `@Transactional(readOnly = true)`.

**Acceptance criteria.** Method annotated; existing tests still pass.

---

## LOW / CLEANUP

### [ ] L1 — Remove redundant `@Autowired`
On the single constructors of `JwtTokenService` and `RefreshTokenService`, remove
`@Autowired` (Spring injects the sole constructor automatically), matching the
other classes' style.
Files: `JwtTokenService.java`, `RefreshTokenService.java`.

### [ ] L2 — Reuse Spring's configured `ObjectMapper`
`AuthSecurityConfig` declares a bare `new ObjectMapper()`. Prefer the
Boot-provided/auto-configured mapper for the JWT serializer, or document why a
dedicated minimal mapper is intentional.
File: `auth/.../config/AuthSecurityConfig.java`.

### [ ] L3 — Validate TTL invariant
In `AuthProperties`, assert `accessTokenTtl` < `refreshTokenTtl` and both are positive.
File: `auth/.../config/AuthProperties.java`.

### [ ] L4 — Fix filename typo
Rename `docer-compose.infra.yml` → `docker-compose.infra.yml`; update any references.

### [ ] L5 — Note ThreadLocal limitation
`TenantContext` uses a `ThreadLocal` that won't propagate across `@Async`/virtual
threads. Add a comment documenting this so future async work accounts for it.
File: `auth/.../security/TenantContext.java`.

---

## TEST GAPS (cross-cutting — address alongside the tasks above)

Current coverage is only `JwtTokenServiceTests` (create + tamper). Add tests for
the security-critical branches:

- [ ] T1 — `JwtTokenService.parse` rejects **expired** tokens and **wrong-issuer** tokens.
- [ ] T2 — `AuthService.validateUserAccess`: inactive user → 403; admin login with
  `platformRole == NONE` → 403; tenant login without active membership → 403.
- [ ] T3 — `AuthService.refresh`: valid rotation issues new tokens and revokes the old;
  invalid/expired/revoked refresh token → 401.
- [ ] T4 — `TenantContextFilter`: non-tenant principal with `X-Organization-Id` → 403;
  invalid UUID → 400; valid membership sets context; no header → passes through.
- [ ] T5 — `JwtAuthenticationFilter.isAllowedForPath`: tenant token on `/api/admin/**`
  is not authenticated; admin token on `/api/**` is not authenticated (cross-area isolation).

---

## Done-definition for the whole file
- All CRITICAL and HIGH tasks completed and tested.
- `./gradlew test` green.
- Security-relevant decisions (H3, H1 topology) recorded in `AGENTS.md`.
