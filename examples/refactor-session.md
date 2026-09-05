# Example session: a pure refactor with nothing to flag

A one-component PR — a method rename plus extracting a helper, no behavior change. Shown to contrast with [`rate-limiter-session.md`](rate-limiter-session.md): not every PR has a finding, and the process doesn't manufacture one to have something to say. Reviewer turns are quoted; everything else is the process's output, following [`output-template.md`](../.apm/skills/guided-pr-review/output-template.md).

---

```
PR REVIEW — Rename CalcTotal to CalculateOrderTotal, extract tax calculation
Branch: refactor/order-total-naming → Base: main
Spec match: None found — reviewing from diff only
Files changed: 1 | Lines: +14 / -8

## Structural Overview
Renames OrderService.CalcTotal to CalculateOrderTotal for clarity and
extracts its tax computation into a new private CalculateTax helper. No
behavior change — the total returned for a given order is identical
before and after.

## Data Flow
Unchanged — the same inputs (order line items) produce the same output
(order total); only internal method names and structure changed.

## Components (1, in call/dependency order)
1. OrderService — src/Services/OrderService.cs — computes an order's
   total, including tax — pure refactor, no behavior change

Walking through each component now.
```

```
### Component 1/1: OrderService
📄 src/Services/OrderService.cs:42
Role: Computes an order's total, including tax; called from the checkout
flow.
Risk: pure refactor, no behavior change
Look for:
- line 42-58: CalculateOrderTotal — confirm the extracted CalculateTax
  helper preserves the exact same rounding and order of operations as the
  original inline calculation
- CalcTotal is public — confirm nothing outside this project calls it by
  name (this repo has no external package consumers, so the rename is safe)
Question: No concerns here — moving on.
```

> The PR's CI runs a naming-convention analyzer that would flag any
> casing violation on its own, so that wasn't worth a line here (see
> `principles.md`, "Defer mechanical checks to existing tooling").

```
## Review Summary
| Component | Finding | Severity | Decision | Notes |
|---|---|---|---|---|
| OrderService | — | — | Approved | Pure refactor; behavior verified unchanged |

Open questions: 0 | Follow-ups filed: 0

The table above is the exact preview of what would post to the PR (plus a "Reviewed by:" line).
Who conducted this review (name or handle)? And: post as-is, add details first, or skip posting?
```

> **Reviewer:** jsmith — skip posting, this one's trivial.

---

This is the fast path through the process: one component, no finding, no severity, no diagram, no pacing flag — every optional piece of the template stayed off because none of it was warranted. The reviewer also chose not to post anything, and the process didn't push back or suggest posting anyway; posting is always the reviewer's call, and "nothing to say" is a legitimate reason to skip it.
