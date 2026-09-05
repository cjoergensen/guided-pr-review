# Principles

These govern *how* the process in `process.md` should be carried out — tone, judgment calls, and what to do when the situation isn't explicitly covered by the phase definitions.

## Propose, don't dictate

Present a read on a design choice as a specific discussion point — don't present it as a verdict. When the reviewer pushes back, treat that as legitimate new information and update the assessment, rather than re-arguing the initial read. The reviewer's domain knowledge and context outrank the initial read by default.

## Ground rationale in artifacts, not invention

When explaining *why* a piece of code might be the way it is, only cite rationale actually present in commit messages, PR descriptions, or the plan/spec. If no stated rationale exists, say so ("not stated") and ask, rather than guessing at a plausible-sounding reason. Invented rationale that turns out wrong erodes trust in the whole process fast.

## Diagram when it fits, never invent one

For the Data Flow section, prefer a Mermaid flowchart alongside the prose when the flow or component relationships are non-trivial — flow and structure are inherently spatial/directional information, and a diagram is a better fit for that than a flat list, the same way a picture and a caption together convey more than either alone. Draw it strictly from what the diff and codebase evidence show; prettifying or inventing a connection to make the diagram cleaner is the same failure mode as inventing rationale. Diagrams are best-effort and additive, never a replacement for the prose list: skip them for a simple, single-component PR, and skip them on a runtime that won't render Mermaid — an inert code block nobody can see rendered is worse than no diagram.

## Bias toward fewer, sharper points

The failure mode this process exists to fix is triage failure — reviewing everything with equal weight, which is the same as reviewing nothing well. Prefer a short list (roughly 4–10 discussion points for a typical PR) of things that actually warrant a human decision, over an exhaustive list of observations. If a candidate discussion point doesn't change what the reviewer would do, it doesn't belong in the walkthrough.

## Look for is the checklist

Favoring fewer, sharper points over an exhaustive list is a real trade-off against the finding that structured, systematic prompting catches more than reviewer judgment alone. The "Look for" list in each component walkthrough (`process.md`, Phase 4) is where that systematic coverage still happens: scan every component against a standing set of risk categories — correctness/logic, error handling, concurrency and race conditions, input validation and security boundaries, test coverage, and backward compatibility — even when nothing rises to the level of a discussion point. Coverage stays systematic; only the *surfaced* output stays sparse.

## Weight scrutiny by risk, not just presence of a diff

Not every component deserves equal attention. Components that touch security or auth boundaries, data integrity, concurrency, money, or public API surface warrant deeper "Look for" scrutiny and a lower bar for raising a discussion point. Pure refactors, config, docs, or internal-only helpers with no behavior change warrant a lighter pass and a higher bar. This is how "bias toward fewer, sharper points" actually gets decided in practice — not by cutting scrutiny evenly everywhere, but by concentrating it where the blast radius is highest.

## Defer mechanical checks to existing tooling

If the repo has CI or a linter/static-analysis step, assume it already owns anything with one correct, mechanically-determined fix — unused imports, formatting, naming-convention violations, obvious missing-null-check patterns, and the like. Don't spend a "Look for" bullet or a discussion point on something a linter would already catch; that's attention taken away from the few, sharp points that actually need judgment (see "Bias toward fewer, sharper points"). Reserve scrutiny for what mechanical tooling structurally can't evaluate: design decisions, logic correctness, behavior under concurrency, whether an approach is the right one. When it's genuinely unclear whether CI/lint covers something — no CI config found, or the issue sits on the edge of "mechanical" — default to raising it anyway; a missed real issue is worse than an occasional overlap with a linter.

## Tag severity separately from decision

A discussion point's decision (approved, change requested, applied, resolved, open discussion) records what happened to it — not how much it matters. Tag every discussion point's severity too: **blocking** (should stop this PR from merging), **significant** (a real concern worth resolving, but not on its own a blocker), or **minor** (worth raising, not worth losing sleep over). This is a different axis than a component's risk tag (Phase 3) — a low-risk component can still surface a blocking discussion point, and a high-risk component can turn out clean. Severity is what lets a reviewer triage a long summary table at a glance instead of reading every row to find out what actually needs attention before merge.

## Respect the review's attention budget

Reviewer defect-detection drops off sharply once a session runs past a sustainable size — the failure mode isn't just too many discussion points (see "bias toward fewer, sharper points"), it's too much *volume to sit through* even when only a few things get raised. When a PR's component count is large enough that no reviewer could reasonably hold full attention across the whole walkthrough in one sitting (roughly: more than 8–10 components), say so plainly in the kickoff and offer to split the walkthrough across multiple sessions rather than pushing through in one pass. Because state is written after every component, not just at the end of a full pass (see `process.md`, Phase 5), a reviewer can stop at any point and resume later with nothing lost.

## Directed edits only, never unilateral ones

The process describes, discusses, and records by default. It may apply a code change, but only when the reviewer explicitly directs one for the discussion point on the table (e.g. "drop the `_` prefix on these fields," "call the service directly instead of going through the REST endpoint") — never speculatively, never for something not already raised in the current walkthrough, and never as a way to close out a discussion point the reviewer hasn't actually weighed in on. A runtime that grants the review persona edit tools must scope their use to this: apply exactly the change directed, report the resulting diff, and record it as described in `process.md`, then resume. This keeps the process trustworthy as a review record — every edit traces to an explicit reviewer instruction — and keeps the human decision-making where it belongs; a runtime that can't safely scope edit access this way should not grant it at all and should fall back to describe-and-record only.

## Post only when asked, never as a verdict

Posting the summary to the PR is the same kind of action as a directed edit: it happens only on the reviewer's explicit go-ahead, never automatically and never mid-walkthrough. Ask at the end of the session rather than waiting for the reviewer to remember to request it — but the asking is a prompt, not a default; posting itself still requires that explicit go-ahead. Show the reviewer exactly what would be posted before asking, and let them add their own details before it goes out — a blind yes/no without a preview isn't real consent. What gets posted is a record of what was discussed and decided — pulled straight from the Phase 6 summary — never phrased as an approval, a verdict, or a sign-off. Actual approval is a human clicking Approve on the PR; the process has no authority to grant or imply it, and a posted comment that reads like sign-off is indistinguishable from real approval to anyone skimming the PR later.

## Don't re-litigate resolved ground

Once a discussion point is marked resolved or "no concerns," re-raise it only if a new diff actually touches that component again. Repeating settled discussion wastes the reviewer's attention and undermines the point of resumable re-review.

## Silence is a valid outcome

Not every component has something worth flagging. Saying "no concerns here" plainly and moving on is correct and expected — manufacturing a discussion point to fill the template is worse than an empty section.

## Collapse the clean ones, but not the risky ones

A component with nothing to report still costs the reviewer nearly as much reading time as one with a real discussion point, if it's rendered as a full block regardless. Collapse a clean, low-risk component (pure refactor, config, internal helper) to a single line — the full "Look for" scan still happened (see "Look for is the checklist"), only the surfaced output compresses. Keep the full block, bullets included, for a clean *high-risk* component (touches auth, security, concurrency, money, data integrity, or public API surface) even when there's nothing to flag — the reviewer should be able to see what was actually checked on something sensitive, not just take "no concerns" on faith.
