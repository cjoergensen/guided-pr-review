# Output Template

The output shape below is fixed — every review, on every PR, on every adapter, follows this skeleton. Consistency of shape is what makes output scannable; only the content inside each section varies.

## Kickoff block

```
PR REVIEW — [PR title]
Branch: [branch] → Base: [base]
Spec match: [Found: specs/NNN-name/ | None found — reviewing from diff only]
Files changed: [n] | Lines: +[x] / -[y]

## Structural Overview
[2-4 sentences: what changed, how components connect]

## Data Flow
[short list: entity in → transform → entity out]

## Components ([n], in call/dependency order)
1. [Component] — [file link] — one-line purpose
2. [Component] — [file link] — one-line purpose
...

Walking through each component now.
```

## Per-component block (repeated once per component, one at a time)

```
### Component [i]/[n]: [Name]
📄 [file link at the relevant line]
Role: [what this component does in the flow]
Look for:
- [specific thing to check, line-level where possible]
- [specific thing to check]
Question: [only if something warrants a decision — otherwise: "No concerns here — moving on."]
[If the reviewer directs a change]: Applied — [what changed] — [file link to the diff/commit]
```

## Re-review block (used instead of the kickoff block when resuming after changes)

```
Re-review — commit [sha]
[n] file(s) changed since last pass: [file list]

### Component [i]/[n]: [Name] — re-checked
[item]: [what was flagged] ✅ resolved | still open | changed — [detail]
...
[If nothing left to flag in this component]: No new concerns in this component. Moving to Component [i+1]/[n].
```

## Summary block (always last)

```
## Review Summary
| Component | Finding | Decision | Notes |
|---|---|---|---|
| ... | ... | Approved / Change requested / Applied / Resolved / Open question | ... |

Open questions: [n] | Follow-ups filed: [n]
```

## Rules for filling the template

- Never skip a section, even if empty — an empty "Data Flow" because the PR is non-functional (e.g. config-only) should say so explicitly rather than omitting the heading.
- The summary table must include every component from the kickoff list, even ones with "No concerns" — completeness of the table is what makes it useful as a record later.
- File links must resolve to an actual line in the current file, not just the file — see the relevant `code-assistants/<tool>/README.md` for the URI scheme that adapter uses.
- An "Applied" decision must link to the actual diff or commit that made the change, not just describe it — and must only appear where the reviewer explicitly directed that edit (see `principles.md`).
