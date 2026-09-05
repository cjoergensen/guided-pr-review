# Example session: a live .tour built while you watch

A small, two-component PR — an in-memory cache in front of product lookups, plus the handler that invalidates it. This example is about the mechanic, not the bug: the reviewer opts into live tours at the very start, and this session actually produces *three* `.tour` files, not one:

- [`live-tour-session-1-productcachedecorator.tour`](live-tour-session-1-productcachedecorator.tour) and [`live-tour-session-2-cacheinvalidationhandler.tour`](live-tour-session-2-cacheinvalidationhandler.tour) — one single-step tour per component, written the moment each is presented, before its discussion point is resolved. These are what the 🧭 links below actually open.
- [`live-tour-session.tour`](live-tour-session.tour) — the whole-PR tour, assembled at Phase 6 once both are resolved, for browsing the finished review in one sequential pass. This is the same shape as [`checkout-pipeline.tour`](checkout-pipeline.tour), the Phase-6 fallback used in [`checkout-pipeline-session.md`](checkout-pipeline-session.md) — the difference here is that the per-component ones already existed *during* the walkthrough.

Below is genuine rendered markdown, not a code block — this is what the process actually outputs, turn by turn. Reviewer turns are quoted and marked with 💬; everything else is the process's own output, following [`output-template.md`](../.apm/skills/guided-pr-review/output-template.md).

---

This looks like a VS Code session — build a live `.tour` file as we go, so you can follow along in the editor? (yes/no)

---

> 💬 **Reviewer:** Yes, let's do that.

---

## PR Review — Add caching to product lookup endpoint

**Branch:** feature/product-cache → **Base:** main\
**Stated intent:** "Adds an in-memory cache in front of ProductRepository.GetById to reduce DB load on the hot path."\
**Spec match:** None found — reviewing from diff only\
**Files changed:** 2 | **Lines:** +58 / -4

### Structural Overview
Adds a decorator that checks an in-memory cache before falling through to the repository, and a handler that evicts the relevant cache entry whenever a product is updated.

### Data Flow
`GetById(id)` → cache hit: return cached value / cache miss: `ProductRepository.GetById(id)`, cache the result → `ProductUpdated` event → `CacheInvalidationHandler` evicts that product's entry

### Components (2, in call/dependency order)
1. **ProductCacheDecorator** — `src/Caching/ProductCacheDecorator.cs` — wraps `IProductRepository`, checks cache before hitting the DB — _new caching layer, correctness-sensitive_
2. **CacheInvalidationHandler** — `src/Caching/CacheInvalidationHandler.cs` — evicts a product's cache entry on update — _concurrency-sensitive, cache correctness_

Walking through each component now.

---

### Component 1/2: ProductCacheDecorator

📄 `src/Caching/ProductCacheDecorator.cs:14`

**Role:** Sits in front of `ProductRepository`; returns the cached value on a hit, otherwise reads through and caches the result.\
**Risk:** _new caching layer, correctness-sensitive_

**Look for:**
- line 18: cache entries have no expiration — eviction relies entirely on `CacheInvalidationHandler` firing

🧭 [Open this component's tour in VS Code](vscode://file//Users/reviewer/repo/.tours/add-caching-to-product-lookup-endpoint/1-productcachedecorator.tour) — see the actual code before weighing in below

**Discussion:** if an invalidation event is ever missed (a crash mid-handler, a dropped message), does anything else bound how stale a cached entry can get?\
**Severity:** 🟡 Significant

---

> 💬 **Reviewer:** Good point — add a 5-minute TTL as a safety net alongside the event-based invalidation. Belt and suspenders.

---

✅ **Applied** — added a 5-minute absolute expiration to each cache entry, alongside the existing event-based eviction — commit `f7a1b2c`

---

### Component 2/2: CacheInvalidationHandler

📄 `src/Caching/CacheInvalidationHandler.cs:9`

**Role:** Handles `ProductUpdated`; evicts that product's entry from the cache.\
**Risk:** _concurrency-sensitive, cache correctness_

**Look for:**
- line 12: eviction happens inside the same handler that processes the update event, with no ordering guarantee relative to the DB transaction that produced it

🧭 [Open this component's tour in VS Code](vscode://file//Users/reviewer/repo/.tours/add-caching-to-product-lookup-endpoint/2-cacheinvalidationhandler.tour) — see the actual code before weighing in below

**Discussion:** is eviction guaranteed to happen *after* the update transaction commits, or could a reader hit the cache and get the old value in the window between commit and eviction?\
**Severity:** 🔴 Blocking

---

> 💬 **Reviewer:** That's a real gap. Publish the event from an outbox after commit, not inline during the transaction, so eviction can never race ahead of the write it's supposed to invalidate.

---

✅ **Applied** — moved `ProductUpdated` publishing to the post-commit outbox dispatcher, so `CacheInvalidationHandler` only ever runs after the write is durable — commit `f7a1b2c`

---

## Review Summary

| Component | Discussion | Severity | Decision | Notes |
|---|---|---|---|---|
| ProductCacheDecorator | No TTL as a fallback if an invalidation event is missed | 🟡 Significant | Applied | Added 5-minute expiration; commit `f7a1b2c` |
| CacheInvalidationHandler | Eviction could race ahead of the commit it's meant to invalidate | 🔴 Blocking | Applied | Moved publish to post-commit outbox; commit `f7a1b2c` |

**Open discussions:** 0 | **Follow-ups filed:** 0

The table above is the exact preview of what would post to the PR (plus a "Reviewed by:" line).

Who conducted this review (name or handle)? And: post as-is, add details first, or skip posting?

All 2 component tours are now also assembled into one whole-PR tour at [.tours/add-caching-to-product-lookup-endpoint.tour](vscode://file//Users/reviewer/repo/.tours/add-caching-to-product-lookup-endpoint.tour), ready to browse in order.

---

> 💬 **Reviewer:** jsmith — skip posting, both fixes are small enough the diff speaks for itself.

---

This session makes two tours do two different jobs, on purpose. The per-component ones exist so the reviewer can look at the actual code *before* forming a view — each is written and linked right after "Look for," before "Discussion," and each is exactly one step, so opening it is unambiguous: no guessing which step a shared multi-step tour happens to be on, no step-jump URI to get wrong. That's also why [`live-tour-session-1-productcachedecorator.tour`](live-tour-session-1-productcachedecorator.tour) and [`live-tour-session-2-cacheinvalidationhandler.tour`](live-tour-session-2-cacheinvalidationhandler.tour) don't carry either fix's resolution — Role, Risk, and the raw discussion question are all that's known at write time.

The whole-PR tour is a different artifact for a different moment: once everything's resolved, Phase 6 assembles [`live-tour-session.tour`](live-tour-session.tour) from both per-component files, now *with* each outcome included, for a teammate who wants to click Next through the entire finished review in order rather than open two separate files. Compare that to [`checkout-pipeline.tour`](checkout-pipeline.tour) — same shape, same purpose, but built as a one-shot fallback in a session where no live tour ran at all.
