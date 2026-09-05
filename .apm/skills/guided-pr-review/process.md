# Review Process

This document defines the review process itself, independent of any specific AI coding assistant. An adapter under `code-assistants/<tool>/` should reference this file rather than re-describing the process.

The unit of review is always **the PR** (or the current branch, if no PR exists yet). Spec/plan context (e.g. from SpecKit) is optional enrichment, not a precondition — the process runs the same way whether or not that context is available, with one branch point in Phase 1.

Output must follow the fixed skeleton in [`output-template.md`](output-template.md). Behavior throughout is governed by [`principles.md`](principles.md) — read that first if anything below seems ambiguous about tone or how firmly to push a point.

---

## Phase 1 — Input resolution

1. Identify the PR or branch being reviewed, and its base.
2. Get the diff against the base — this is always the primary input, regardless of everything else.
3. **Best-effort spec lookup**: check whether the branch/changed files correspond to a spec folder (e.g. SpecKit's `specs/<NNN-feature-name>/` convention) and, if so, load `spec.md`, `plan.md`, and `tasks.md`. Match changed files to task IDs where possible.
   - **If found**: subsequent phases include a plan-vs-implementation comparison.
   - **If not found**: subsequent phases run structural/risk-based review only. This is not an error state — most PRs may not map to a spec, and the process should treat this as the normal case, not a degraded one.
4. Check for an existing `review-notes.md` in the PR/spec folder from a prior pass (see Phase 5). If present, load it to reconstruct what's already resolved before diffing new commits.

## Phase 2 — Structural overview

Produce a short (2–4 sentence) description of:
- What changed, at the level of components/modules, not files.
- How the changed pieces connect to each other and to existing code.
- The data flow through the change: entity in → transformation(s) → entity out.

Keep this brief. It orients the reviewer before the walkthrough — it is not the review itself.

## Phase 3 — Component inventory

List the components/services/modules touched by this PR, **in call or dependency order** (the order a request or data actually flows through them), not file-alphabetical or diff order. For each: name, primary file path, and a one-line purpose.

This list becomes the walkthrough order in Phase 4 and the row order in the Phase 6 summary.

## Phase 4 — Component walkthrough

Present components **one at a time**, waiting for reviewer input between each. For each component:

- **File link**: point directly at the relevant file and line, using the file-link mechanism appropriate to the adapter (e.g. `vscode://file/{absolute-path}:{line}` for VS Code–based tools). Adapters are responsible for supplying the correct scheme.
- **Role**: what this component does in the overall flow (one line).
- **Look for**: specific, concrete things to check in this component — line-level where possible (e.g. "line 58: token refill logic — verify no race under concurrent requests"), not generic advice like "check for bugs."
- **Plan comparison** (only if a spec was found in Phase 1): note any deviation between what `plan.md` describes and what this component actually does.
- **Question**: only when something is genuinely worth a decision. If nothing rises to that bar, say so plainly ("No concerns here — moving on.") rather than manufacturing a question.

A reviewer's response to a component may approve it, answer or push back on the question, or **direct a specific change** (e.g. "drop the `_` prefix on these fields," "call the service directly instead of the REST endpoint"). Only in that last case, and only on adapters that grant edit tools (see `principles.md`, "Directed edits only, never unilateral ones"), apply exactly the change described, report the resulting diff, and record it per Phase 5. Never apply a change that wasn't explicitly directed for the finding at hand, and never use a directed edit as license to touch anything beyond it.

Move to the next component only after the reviewer has responded to the current one and any directed edit has been applied and confirmed.

## Phase 5 — Feedback capture and resumable re-review

For every point raised in Phase 4, record: the finding, the decision (approved / change requested / applied / open question), and the reasoning — including the reviewer's own argument if they pushed back on the initial read. For "applied," the reasoning is the reviewer's directive itself, and the record must reference the resulting diff or commit.

After each pass (initial or re-review), write this state to `review-notes.md` in the PR or spec folder, in the same shape as the Phase 6 summary table.

Edits applied directly during the walkthrough are already reflected in the working tree and don't need a separate re-review pass for that same finding — but any further commits or pushes still go through the resumable flow below.

**When the reviewer reports changes have been made** (a new commit, a push):
1. Diff only the files changed since the last recorded pass.
2. For each component with an *open or change-requested* item, re-check specifically that item against the new code and report resolved / still-open / newly-changed.
3. Do **not** re-litigate components already marked "no concerns" or "resolved" unless the new diff actually touches them.
4. Continue the walkthrough from wherever it left off — do not restart from Component 1 unless the reviewer asks to.

## Phase 6 — Summary

Always close with the fixed summary table from `output-template.md`: one row per component/finding, its decision, and any notes. Include counts of open questions and follow-ups filed.
