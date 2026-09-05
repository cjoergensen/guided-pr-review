# Example session: adding rate limiting to an API endpoint

A small, two-component PR — a token-bucket rate limiter and the ASP.NET Core middleware that wires it into request handling. No spec/plan folder for this one, so it's a structural/risk-based review. Reviewer turns are quoted; everything else is the process's output, following [`output-template.md`](../.apm/skills/guided-pr-review/output-template.md).

---

```
PR REVIEW — Add rate limiting to /api/orders
Branch: feat/rate-limit-orders → Base: main
Spec match: None found — reviewing from diff only
Files changed: 2 | Lines: +91 / -3

## Structural Overview
Adds a token-bucket rate limiter and a middleware that applies it to the
orders endpoint. The middleware sits in front of the existing handler and
short-circuits with a 429 before the request reaches it.

## Data Flow
Incoming request → RateLimitMiddleware (checks bucket, consumes a token) →
TokenBucket (refill + consume) → allowed: forwards to next() / denied: 429

## Components (2, in call/dependency order)
1. RateLimitMiddleware — src/Api/Middleware/RateLimitMiddleware.cs — applies
   the limiter to incoming requests — touches request handling, low complexity
2. TokenBucket — src/RateLimiting/TokenBucket.cs — refill/consume algorithm —
   new public API surface, concurrency-sensitive

Walking through each component now.
```

```
### Component 1/2: RateLimitMiddleware
📄 src/Api/Middleware/RateLimitMiddleware.cs:24
Role: Intercepts incoming requests via InvokeAsync, looks up the caller's
bucket, and either calls next() or short-circuits with 429.
Risk: touches request handling, low complexity
Look for:
- line 31: bucket key derivation — confirm it's per-user, not global
- line 42: 429 response sets a Retry-After header
Question: is the bucket keyed per authenticated user, or is there a risk
it's effectively one global bucket for all callers?
Severity: significant
```

> **Reviewer:** Keyed per-user — `var key = context.User.FindFirstValue(ClaimTypes.NameIdentifier);`
> on line 31, pulled from the auth middleware that already runs before this
> one. Confirmed correct.

```
### Component 2/2: TokenBucket
📄 src/RateLimiting/TokenBucket.cs:14
Role: Holds the refill/consume state for one bucket; the middleware calls
`Consume()` once per request.
Risk: new public API surface, concurrency-sensitive
Look for:
- line 18-25: Refill() math — tokens accrue correctly over elapsed time
- line 27: Consume() reads and mutates `_tokens` with no lock
Question: this app serves requests from the ASP.NET Core thread pool — is
there a race between two concurrent requests both calling Consume() on the
same bucket instance?
Severity: blocking
```

> **Reviewer:** Good catch, yeah — under load two threads could both read
> the same token count before either decrements it. Wrap Refill() and
> Consume() in a lock.

```
Applied — added a private readonly object _lock and wrapped Refill() and
Consume() in a lock (_lock) { ... } block — a1b2c3d
```

```
## Review Summary
| Component | Finding | Severity | Decision | Notes |
|---|---|---|---|---|
| RateLimitMiddleware | Bucket keying strategy (per-user vs global) | Significant | Approved | Confirmed keyed by `ClaimTypes.NameIdentifier` |
| TokenBucket | No lock around Refill/Consume under concurrent requests | Blocking | Applied | Added `lock (_lock) { ... }`; commit a1b2c3d |

Open questions: 0 | Follow-ups filed: 0

The table above is the exact preview of what would post to the PR (plus a "Reviewed by:" line).
Who conducted this review (name or handle)? And: post as-is, add details first, or skip posting?
```

> **Reviewer:** jsmith — post as-is.

```
Posted to PR #482:

## Review Summary
| Component | Finding | Severity | Decision | Notes |
|---|---|---|---|---|
| RateLimitMiddleware | Bucket keying strategy (per-user vs global) | Significant | Approved | Confirmed keyed by `ClaimTypes.NameIdentifier` |
| TokenBucket | No lock around Refill/Consume under concurrent requests | Blocking | Applied | Added `lock (_lock) { ... }`; commit a1b2c3d |

Open questions: 0 | Follow-ups filed: 0
Reviewed by: jsmith
```

---

Two things this session shows in miniature: the concurrency question on `TokenBucket` was severity **blocking** and got fixed inline via a directed edit, while the keying question on `RateLimitMiddleware` was **significant** but resolved by the reviewer's own explanation — no code changed, because nothing was actually wrong. Both are valid outcomes; the process never manufactured a fix where an answer was enough.
