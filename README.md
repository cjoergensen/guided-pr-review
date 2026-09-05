# guided-pr-review

A guided, component-by-component peer-review **facilitator** for AI coding assistants — it walks a human reviewer through a PR one piece at a time; it does not review the PR *for* them.

## Facilitator, not reviewer

This doesn't approve or reject anything, and it never renders a verdict — every decision in the summary table is the human's, not the process's. Its job is making sure a human reviewer sees the right things, in the right order, at the right depth, and never has to re-litigate settled ground — not replacing the judgment call itself. It never edits code on its own either: a change only happens when the human explicitly directs it for a specific finding (see [`principles.md`](.apm/skills/guided-pr-review/principles.md), "Directed edits only, never unilateral ones").

## The problem

File-by-file review of AI-generated (or any) implementations is exhausting and low-value: it surfaces line noise, not design decisions, and it gives no sense of how the pieces fit together before you're three files deep.

## What this does instead

- **Anchors on the PR**, not on a spec — every PR gets walked through the same way.
- **Uses spec/plan context when it exists** (e.g. from SpecKit) to check implementation against stated intent — this is the actual differentiator over a generic AI reviewer. When no spec is found, it falls back to structural + risk-based review from the diff and codebase alone.
- **Gives a structural overview first** — what changed, how components connect, the data flow through the change — before touching any code.
- **Walks through components one at a time**, in call/dependency order, linking directly to the relevant file and line, and pointing out specifically what to look at and why.
- **Asks, rather than dictates** — surfaces its read on a design choice and a specific question, then waits. Pushback changes its assessment; it doesn't argue past you.
- **Captures the decision and the reasoning**, not just a verdict, so "why we built it this way" survives past the conversation.
- **Resumes from where it left off** — after you push a fix, it re-checks only what changed and carries forward everything already resolved, rather than restarting the review.
- **Same output shape every time** — a fixed template (see [`output-template.md`](.apm/skills/guided-pr-review/output-template.md)) so review output is predictable and scannable regardless of what's in the diff.

## Examples

- [`examples/refactor-session.md`](examples/refactor-session.md) — a one-component pure refactor (a rename plus an extracted helper, no behavior change) with nothing to flag. Shows the process doesn't manufacture a question just to have something to say.
- [`examples/rate-limiter-session.md`](examples/rate-limiter-session.md) — a small, two-component PR: a token-bucket rate limiter plus the middleware wiring it into request handling. One question resolves with no code change; the other surfaces a real concurrency bug and gets fixed inline via a directed edit.
- [`examples/checkout-pipeline-session.md`](examples/checkout-pipeline-session.md) — a larger, six-component checkout pipeline with a genuinely branching flow, where the optional Mermaid diagram earns its place. Covers every decision state and severity tier in one summary table, plus the "add details" branch of the closing post-to-PR flow.

## Status

Early work in progress. Core process is designed and packaged as an [APM](https://microsoft.github.io/apm/) skill; Copilot and Claude Code deploys are validated, Cursor is next.

## Repository structure

```
apm.yml                              # package manifest — name, version, targets, deps
.apm/
├── skills/
│   └── guided-pr-review/
│       ├── SKILL.md                 # entry point every APM target discovers
│       ├── process.md               # the phases: input resolution, overview,
│       │                            #   component walkthrough, feedback capture,
│       │                            #   resumable re-review
│       ├── output-template.md       # the fixed output skeleton
│       └── principles.md            # behavioral rules (propose don't dictate,
│                                     #   ground in artifacts, directed edits only)
└── prompts/
    └── review-pr.prompt.md          # Copilot-native `/review-pr` slash command

examples/                            # worked example sessions, for reference
├── refactor-session.md              # simplest case: nothing to flag
├── rate-limiter-session.md          # small case: one bug, one non-issue
└── checkout-pipeline-session.md     # larger case: diagram, full state coverage
```

`SKILL.md`, `process.md`, `output-template.md`, and `principles.md` are the single source of truth — nothing duplicates them elsewhere. `apm install` hoists the `skills` and `prompts` primitives into whichever native location each target expects (`.agents/skills/guided-pr-review/` for Copilot and most other targets, `.claude/skills/` for Claude Code, `.github/prompts/` for Copilot's `/review-pr` command) — there's no hand-written per-tool adapter to keep in sync.

## Using it with GitHub Copilot

```bash
apm install cjoergensen/guided-pr-review --target copilot
apm compile
```

Then run `/review-pr` from a Copilot Chat session on the branch or PR you want reviewed, or just ask Copilot to review the PR — it can also discover the `guided-pr-review` skill directly.

## Using it with Claude Code

```bash
apm install cjoergensen/guided-pr-review --target claude
```

The `prompts` primitive has no Claude equivalent, so `apm install` converts it automatically into a `/review-pr` command (`.claude/commands/review-pr.md`), alongside the skill at `.claude/skills/guided-pr-review/`. Run `/review-pr` from Claude Code, or just ask it to review the PR.

## Roadmap

- [x] Validate the Copilot skill + prompt deploy end-to-end
- [x] Validate the Claude Code skill + command deploy end-to-end
- [ ] Add `cursor` to `targets:` in `apm.yml` once verified on Cursor

## License

MIT — see [LICENSE](LICENSE).
