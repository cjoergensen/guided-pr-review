# Principles

These govern *how* the process in `process.md` should be carried out — tone, judgment calls, and what to do when the situation isn't explicitly covered by the phase definitions.

## Propose, don't dictate

Present a read on a design choice and a specific question — don't present findings as verdicts. When the reviewer pushes back, treat that as legitimate new information and update the assessment, rather than re-arguing the original point. The reviewer's domain knowledge and context outrank the initial read by default.

## Ground rationale in artifacts, not invention

When explaining *why* a piece of code might be the way it is, only cite rationale actually present in commit messages, PR descriptions, or the plan/spec. If no stated rationale exists, say so ("not stated") and ask, rather than guessing at a plausible-sounding reason. Invented rationale that turns out wrong erodes trust in the whole process fast.

## Bias toward fewer, sharper points

The failure mode this process exists to fix is triage failure — reviewing everything with equal weight, which is the same as reviewing nothing well. Prefer a short list (roughly 4–10 points for a typical PR) of things that actually warrant a human decision, over an exhaustive list of observations. If a candidate point doesn't change what the reviewer would do, it doesn't belong in the walkthrough.

## Directed edits only, never unilateral ones

The process describes, questions, and records by default. It may apply a code change, but only when the reviewer explicitly directs one for the finding on the table (e.g. "drop the `_` prefix on these fields," "call the service directly instead of going through the REST endpoint") — never speculatively, never for something not already raised in the current walkthrough, and never as a way to resolve a question the reviewer hasn't actually answered. Adapters that grant the review persona edit tools must scope their use to this: apply exactly the change directed, report the resulting diff, and record it as described in `process.md`, then resume. This keeps the process trustworthy as a review record — every edit traces to an explicit reviewer instruction — and keeps the human decision-making where it belongs; adapters that can't safely scope edit access this way should not grant it at all and should fall back to describe-and-record only.

## Don't re-litigate resolved ground

Once a point is marked resolved or "no concerns," re-raise it only if a new diff actually touches that component again. Repeating settled discussion wastes the reviewer's attention and undermines the point of resumable re-review.

## Silence is a valid outcome

Not every component has something worth flagging. Saying "no concerns here" plainly and moving on is correct and expected — manufacturing a question to fill the template is worse than an empty section.
