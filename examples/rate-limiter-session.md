# Example session: adding rate limiting to an API endpoint

A small, two-component PR — a token-bucket rate limiter and the ASP.NET Core middleware that wires it into request handling. No spec/plan folder for this one, so it's a structural/risk-based review.

Below is genuine rendered markdown, not a code block — this is what the process actually outputs, turn by turn, in a real session. Reviewer turns are quoted and marked with 💬 so they stand out from the process's own bold-heavy formatting; everything else is the process's own output, following [`output-template.md`](../.apm/skills/guided-pr-review/output-template.md).

---

## PR Review — Add rate limiting to /api/orders

**Branch:** feat/rate-limit-orders → **Base:** main\
**Stated intent:** "Adds per-user rate limiting to /api/orders to stop a single client from hammering the endpoint."\
**Spec match:** None found — reviewing from diff only\
**Files changed:** 2 | **Lines:** +91 / -3

### Structural Overview
Adds a token-bucket rate limiter and a middleware that applies it to the orders endpoint. The middleware sits in front of the existing handler and short-circuits with a 429 before the request reaches it.

### Data Flow
Incoming request → `RateLimitMiddleware` (checks bucket, consumes a token) → `TokenBucket` (refill + consume) → allowed: forwards to `next()` / denied: 429

### Components (2, in call/dependency order)
1. **RateLimitMiddleware** — `src/Api/Middleware/RateLimitMiddleware.cs` — applies the limiter to incoming requests — _touches request handling, low complexity_
2. **TokenBucket** — `src/RateLimiting/TokenBucket.cs` — refill/consume algorithm — _new public API surface, concurrency-sensitive_

Walking through each component now.

---

### Component 1/2: RateLimitMiddleware

📄 `src/Api/Middleware/RateLimitMiddleware.cs:24`

**Role:** Intercepts incoming requests via `InvokeAsync`, looks up the caller's bucket, and either calls `next()` or short-circuits with 429.\
**Risk:** _touches request handling, low complexity_

**Look for:**
- line 31: bucket key derivation — confirm it's per-user, not global
- line 42: 429 response sets a `Retry-After` header

**Discussion:** is the bucket keyed per authenticated user, or is there a risk it's effectively one global bucket for all callers?\
**Severity:** 🟡 Significant

---

> 💬 **Reviewer:** Keyed per-user — `var key = context.User.FindFirstValue(ClaimTypes.NameIdentifier);` on line 31, pulled from the auth middleware that already runs before this one. Confirmed correct.

---

### Component 2/2: TokenBucket

📄 `src/RateLimiting/TokenBucket.cs:14`

**Role:** Holds the refill/consume state for one bucket; the middleware calls `Consume()` once per request.\
**Risk:** _new public API surface, concurrency-sensitive_

**Look for:**
- line 18-25: `Refill()` math — tokens accrue correctly over elapsed time
- line 27: `Consume()` reads and mutates `_tokens` with no lock

**Discussion:** this app serves requests from the ASP.NET Core thread pool — is there a race between two concurrent requests both calling `Consume()` on the same bucket instance?\
**Severity:** 🔴 Blocking

---

> 💬 **Reviewer:** Good catch, yeah — under load two threads could both read the same token count before either decrements it. Wrap `Refill()` and `Consume()` in a lock.

---

✅ **Applied** — added a private `readonly object _lock` and wrapped `Refill()` and `Consume()` in a `lock (_lock) { ... }` block — commit `a1b2c3d`

---

## Review Summary

| Component | Discussion | Severity | Decision | Notes |
|---|---|---|---|---|
| RateLimitMiddleware | Bucket keying strategy (per-user vs global) | 🟡 Significant | Approved | Confirmed keyed by `ClaimTypes.NameIdentifier` |
| TokenBucket | No lock around Refill/Consume under concurrent requests | 🔴 Blocking | Applied | Added `lock (_lock) { ... }`; commit `a1b2c3d` |

**Open discussions:** 0 | **Follow-ups filed:** 0

The table above is the exact preview of what would post to the PR (plus a "Reviewed by:" line).

Who conducted this review (name or handle)? And: post as-is, add details first, or skip posting?

---

> 💬 **Reviewer:** jsmith — post as-is.

---

**Posted to PR #482:**

## Review Summary

| Component | Discussion | Severity | Decision | Notes |
|---|---|---|---|---|
| RateLimitMiddleware | Bucket keying strategy (per-user vs global) | 🟡 Significant | Approved | Confirmed keyed by `ClaimTypes.NameIdentifier` |
| TokenBucket | No lock around Refill/Consume under concurrent requests | 🔴 Blocking | Applied | Added `lock (_lock) { ... }`; commit `a1b2c3d` |

**Open discussions:** 0 | **Follow-ups filed:** 0

**Reviewed by:** jsmith

---

Two things this session shows in miniature: the concurrency discussion point on `TokenBucket` was severity **blocking** — the 🔴 marker makes it findable at a glance in the table above — and got fixed inline via a directed edit, while the keying discussion point on `RateLimitMiddleware` was **significant** (🟡) but resolved by the reviewer's own explanation — no code changed, because nothing was actually wrong. Both are valid outcomes; the process never manufactured a fix where an answer was enough.
