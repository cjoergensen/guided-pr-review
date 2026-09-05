# Review Process

This document defines the review process itself, independent of any specific AI coding assistant. It's distributed as an APM skill (see the accompanying `SKILL.md`) so any target that supports the `skills` primitive gets this file as-is rather than a re-description of the process.

The unit of review is always **the PR** (or the current branch, if no PR exists yet). Spec/plan context (e.g. from SpecKit) is optional enrichment, not a precondition — the process runs the same way whether or not that context is available, with one branch point in Phase 1.

Output must follow the fixed skeleton in [`output-template.md`](output-template.md). Behavior throughout is governed by [`principles.md`](principles.md) — read that first if anything below seems ambiguous about tone or how firmly to push a point.

---

## Phase 1 — Input resolution

1. Identify the PR or branch being reviewed, and its base.
2. Get the diff against the base — this is always the primary input, regardless of everything else.
3. **Note the PR's own stated intent**: its title and description, when available — this is an artifact like any other (see `principles.md`, "Ground rationale in artifacts, not invention"), and it's present far more often than a formal spec. Phase 2 uses it to flag, without inventing motive, anything the diff does that the description doesn't account for.
4. **Best-effort spec lookup**: check whether the branch/changed files correspond to a spec folder (e.g. SpecKit's `specs/<NNN-feature-name>/` convention) and, if so, load `spec.md`, `plan.md`, and `tasks.md`. Match changed files to task IDs where possible.
   - **If found**: subsequent phases include a plan-vs-implementation comparison.
   - **If not found**: subsequent phases run structural/risk-based review only. This is not an error state — most PRs may not map to a spec, and the process should treat this as the normal case, not a degraded one.
5. Check for an existing `review-notes.md` in the PR/spec folder from a prior pass (see Phase 5). If present, load it to reconstruct what's already resolved before diffing new commits.
6. Check for CI and/or a linter/static-analysis config (a CI workflow file, an `.eslintrc`/`ruff`/`pylintrc`-style config, a pre-commit hook, etc.). Its presence determines whether Phase 4 defers mechanical checks to it (see `principles.md`, "Defer mechanical checks to existing tooling") or, absent any such tooling, still raises them itself.

## Phase 2 — Structural overview

Produce a short (2–4 sentence) description of:
- What changed, at the level of components/modules, not files.
- How the changed pieces connect to each other and to existing code.
- The data flow through the change: entity in → transformation(s) → entity out.

Keep this brief. It orients the reviewer before the walkthrough — it is not the review itself.

If the diff does something the PR's stated intent (Phase 1) doesn't account for, say so explicitly here — plainly, without guessing at why. This is the same comparison a formal plan-vs-implementation check does, just running on the one artifact (title + description) that's almost always available instead of the one that usually isn't.

When the data flow or component relationships are non-trivial and the runtime renders Mermaid, add a `flowchart` diagram alongside the prose, not instead of it (see `principles.md`, "Diagram when it fits, never invent one"). Skip it for a simple, single-component PR, or on a runtime that won't render it — the prose always stands on its own.

## Phase 3 — Component inventory

List the components/services/modules touched by this PR, **in call or dependency order** (the order a request or data actually flows through them), not file-alphabetical or diff order. For each: name, primary file path, a one-line purpose, and a brief risk tag (e.g. "touches auth," "pure refactor, no behavior change," "new public API surface," "internal helper") — Phase 4 uses this to calibrate how much scrutiny the component gets (see `principles.md`, "Weight scrutiny by risk").

This list becomes the walkthrough order in Phase 4 and the row order in the Phase 6 summary.

If the component count is large enough that no reviewer could reasonably sustain full attention across the whole walkthrough in one sitting (roughly: more than 8–10 components), say so in the kickoff block and offer to split the walkthrough across multiple sessions (see `principles.md`, "Respect the review's attention budget").

## Phase 4 — Component walkthrough

Present components **one at a time**, waiting for reviewer input between each. For each component:

- **File link**: point directly at the relevant file and line, using whatever file-link scheme the runtime supports (e.g. `vscode://file/{absolute-path}:{line}` for VS Code–based tools).
- **Role**: what this component does in the overall flow (one line).
- **Look for**: specific, concrete things to check in this component — line-level where possible (e.g. "line 58: token refill logic — verify no race under concurrent requests"), not generic advice like "check for bugs." Scan against the standing risk categories in `principles.md` ("Look for is the checklist") for every component, but surface only what's actually worth checking — coverage is systematic even where the output stays sparse. Depth here scales with the component's risk tag from Phase 3: more scrutiny for auth/security/concurrency/money/public-API components, a lighter pass for pure refactors, config, or internal-only helpers. If Phase 1 found CI/lint tooling in place, skip anything with one mechanically-determined fix — unused imports, formatting, naming conventions — and spend the slot on design and logic instead (see `principles.md`, "Defer mechanical checks to existing tooling").
- **Plan comparison** (only if a spec was found in Phase 1): note any deviation between what `plan.md` describes and what this component actually does.
- **Discussion**: only when something is genuinely worth a decision. Tag its severity — blocking, significant, or minor (see `principles.md`, "Tag severity separately from decision") — alongside it. If nothing rises to that bar, say so plainly ("No concerns here — moving on.") rather than manufacturing a discussion point.

When a component turns up nothing worth a discussion point after the full "Look for" scan, render it as a single collapsed line instead of the full block — the scan still happened; only the surfaced output compresses. Reserve the full block for a clean but high-risk component too (auth, security, concurrency, money, data integrity, public API) so the reviewer can see what was actually checked, not just take "no concerns" on faith (see `principles.md`, "Collapse the clean ones, but not the risky ones").

A reviewer's response to a component may approve it, weigh in on the discussion point, or **direct a specific change** (e.g. "drop the `_` prefix on these fields," "call the service directly instead of the REST endpoint"). Only in that last case, and only on a runtime that grants edit tools (see `principles.md`, "Directed edits only, never unilateral ones"), apply exactly the change described, report the resulting diff, and record it per Phase 5. Never apply a change that wasn't explicitly directed for the discussion point at hand, and never use a directed edit as license to touch anything beyond it. If the runtime doesn't grant edit tools, or the reviewer directs a change without asking for it inline, record the decision as "change requested" instead of "applied," and note in Phase 5 which of those two reasons it was — a later reader needs to know whether the fix still has to happen.

Move to the next component only after the reviewer has responded to the current one and any directed edit has been applied and confirmed.

## Phase 5 — Feedback capture and resumable re-review

For every discussion point raised in Phase 4, record: what was raised, its severity (blocking / significant / minor), the decision (approved / change requested / applied / open discussion), and the reasoning — including the reviewer's own argument if they pushed back on the initial read. For "applied," the reasoning is the reviewer's directive itself, and the record must reference the resulting diff or commit.

Write this state to `review-notes.md` in the PR or spec folder **after every component**, not only at the end of a full pass — this is what makes it safe for a reviewer to stop mid-walkthrough (see `principles.md`, "Respect the review's attention budget") and resume later with nothing lost, whether or not a new commit has been pushed in between. Use the same shape as the Phase 6 summary table.

Edits applied directly during the walkthrough are already reflected in the working tree and don't need a separate re-review pass for that same discussion point — but any further commits or pushes still go through the resumable flow below.

**When the reviewer reports changes have been made** (a new commit, a push):
1. Diff only the files changed since the last recorded pass.
2. For each component with an *open-discussion or change-requested* item, re-check specifically that item against the new code, carrying its original severity forward, and report resolved / still-open / newly-changed — a still-open blocking item and a still-open minor one are not the same news, and the re-review output should say so.
3. Do **not** re-litigate components already marked "no concerns" or "resolved" unless the new diff actually touches them.
4. Continue the walkthrough from wherever it left off — do not restart from Component 1 unless the reviewer asks to.

## Phase 6 — Summary

Always close with the fixed summary table from `output-template.md`: one row per component/discussion point, its decision, and any notes. Include counts of open discussions and follow-ups filed.

End the session by showing this table as the exact preview of what would be posted, and ask who conducted the review (name or handle, for attribution — don't invent or assume one) and whether to post it as-is, add details first, or skip posting entirely. Don't wait for the reviewer to remember to request it.

- **Post as-is**: post a single comment containing the table, counts, and a "Reviewed by: [name]" line — nothing else, and no language implying approval, a verdict, or team sign-off (see `principles.md`, "Post only when asked, never as a verdict").
- **Add details**: take the reviewer's own free text, append it under a distinct "Additional notes" section below the table, then post — the added text is the reviewer's own words, not a restatement or embellishment of the discussion above.
- **Skip**: don't post anything.

Never post automatically; posting always requires that explicit choice, the same as a directed edit.
