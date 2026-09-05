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
- **Asks, rather than dictates** — surfaces its read on a design choice as a specific discussion point, then waits. Pushback changes its assessment; it doesn't argue past you.
- **Captures the decision and the reasoning**, not just a verdict, so "why we built it this way" survives past the conversation.
- **Resumes from where it left off** — after you push a fix, it re-checks only what changed and carries forward everything already resolved, rather than restarting the review.
- **Same output shape every time** — a fixed template (see [`output-template.md`](.apm/skills/guided-pr-review/output-template.md)) so review output is predictable and scannable regardless of what's in the diff.
- **Can build [CodeTour](https://marketplace.visualstudio.com/items?itemName=vsls-contrib.codetour) `.tour` files as it goes**, on VS Code — asked once, up front. Each component gets its own single-step tour, written and linked the moment it's presented, so you can look at the actual code before weighing in, not after. Once the walkthrough ends, those get assembled into one whole-PR tour for browsing the finished review in order — or, if you skipped the up-front ask, a one-shot version of that same whole-PR tour is still offered at the end. Either way, always opt-in, always separate from posting to the PR.

## Grounded in research

Most of the design choices above trace back to a specific finding, not a hunch. Grouped by what they justify:

**Review methodology**
- **Fagan's original 1976 code inspection method** — the origin of structured, complete-record review (vs. ad hoc reading). Justifies the fixed output template, the rule that a "no concerns" component still gets a row in the summary table, and the severity/decision taxonomies.
- **Rigby & Bird, *Convergent Contemporary Software Peer Review Practices* (2013)** — async, tool-mediated review achieves comparable defect detection to Fagan's heavyweight inspection meetings, at a fraction of the overhead. Justifies running as a chat-based, resumable process rather than a synchronous review meeting.
- **Sadowski et al., *Modern Code Review: A Case Study at Google* (ICSE SEIP 2018)** — not understanding a change's purpose before reading it is the top-cited source of review friction. Justifies the Structural Overview and comparing the diff against the PR's stated intent even without a formal spec.
- **Bacchelli & Bird, *Expectations, Outcomes, and Challenges of Modern Code Review* (ICSE 2013, Microsoft)** — verifying intent and knowledge-sharing are reported ahead of pure defect-hunting as reasons people review at all. Justifies capturing the decision *and* the reasoning, not just a verdict.
- **Cohen et al. / the SmartBear–Cisco *Best Kept Secrets of Peer Code Review* study** — reviewer defect-detection drops off sharply past a sustainable session size. Justifies the pacing threshold and one-component-at-a-time presentation.
- **MacLeod et al., *Code Reviewing in the Trenches* (IEEE Software 2018)** — "finding the code being discussed" is a recurring tooling complaint. Justifies deep-linking every component to its exact file and line.
- **Egelman et al., *Predicting Developers' Negative Feelings about Code Review* (ICSE 2020)** — nitpick volume and tone correlate with reviewer/author fatigue and disengagement. Justifies biasing toward fewer, sharper discussion points and treating "no concerns" as a valid, expected outcome.
- **Review-iteration studies on real Gerrit/Chromium data** — each additional full re-review round measurably adds latency and fatigue. Justifies delta-only, resumable re-review that never re-litigates settled components.
- **Potts & Bruns, design rationale research (1988)** — undocumented decisions get re-litigated by people who weren't there the first time. Justifies recording *why*, including the reviewer's own pushback, alongside every decision.

**Cognitive science of presentation**
- **Paivio's dual-coding theory** — information encoded both verbally and visually integrates into stronger comprehension than either alone. Justifies the optional Mermaid diagram alongside (never instead of) the prose Data Flow list.
- **Vessey's Cognitive Fit Theory (1991)** — a representation works best when its structure matches the task's structure; flow and dependency relationships are inherently spatial/directional. Justifies drawing a diagram specifically for flow and component relationships rather than forcing them into a flat list.
- **Pennington; von Mayrhauser & Vans, program comprehension research** — building a "what does this do" model before a "how does it do it" model produces more accurate mental models. Justifies the structural overview before any line-level detail, and walking components in call/dependency order rather than diff or alphabetical order.

**Document and interface design**
- **Nielsen's "Consistency and Standards" usability heuristic (1994)** and **the "jingle-jangle fallacy" (Kelley, 1927)** — using different words for the same concept forces readers to reconcile them as separate things. Justifies unifying what were once three different names ("Question," "Finding," "point") into one canonical term, "discussion point," everywhere.
- **Hartley's structured-abstracts research** — a fixed, labeled format is read faster and recalled better than equivalent free-form prose, but only if the same label reliably means the same thing everywhere. Justifies both the fixed template itself and the terminology consolidation above.
- **Alarm-fatigue research (clinical and security alerting literature)** — a warning shown regardless of whether it's warranted trains people to ignore it. Justifies the pacing flag appearing only when the component count actually crosses the threshold, never by default.

## Examples

- [`examples/refactor-session.md`](examples/refactor-session.md) — a one-component pure refactor (a rename plus an extracted helper, no behavior change) with nothing to flag. Shows the process doesn't manufacture a discussion point just to have something to say.
- [`examples/rate-limiter-session.md`](examples/rate-limiter-session.md) — a small, two-component PR: a token-bucket rate limiter plus the middleware wiring it into request handling. One discussion point resolves with no code change; the other surfaces a real concurrency bug and gets fixed inline via a directed edit.
- [`examples/checkout-pipeline-session.md`](examples/checkout-pipeline-session.md) — a larger, six-component checkout pipeline with a genuinely branching flow, where the optional Mermaid diagram earns its place. Covers every decision state and severity tier in one summary table, the "add details" branch of the closing post-to-PR flow, and the Phase-6 fallback `.tour` export ([`checkout-pipeline.tour`](examples/checkout-pipeline.tour) is the actual generated file).
- [`examples/live-tour-session.md`](examples/live-tour-session.md) — a small, two-component PR where the reviewer opts into live tours up front instead. Each component gets its own single-step `.tour` file the moment it's presented ([`live-tour-session-1-productcachedecorator.tour`](examples/live-tour-session-1-productcachedecorator.tour), [`live-tour-session-2-cacheinvalidationhandler.tour`](examples/live-tour-session-2-cacheinvalidationhandler.tour)), then both get assembled into one whole-PR tour at the end ([`live-tour-session.tour`](examples/live-tour-session.tour)) — two tours doing two different jobs.

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
├── checkout-pipeline-session.md     # larger case: diagram, full state coverage
├── checkout-pipeline.tour           # its Phase-6 fallback whole-PR .tour
├── live-tour-session.md             # live tours: per-component, then assembled
├── live-tour-session-1-*.tour       # component 1's own single-step tour
├── live-tour-session-2-*.tour       # component 2's own single-step tour
└── live-tour-session.tour           # both assembled into one whole-PR tour
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
- [x] Optional `.tour` export, two tours for two jobs — one single-step tour per component, written live as each is presented (so the reviewer can view the code before weighing in), assembled at the end into one whole-PR tour for sequential browsing; a Phase-6 one-shot fallback covers reviewers who skipped the up-front ask ([`checkout-pipeline.tour`](examples/checkout-pipeline.tour), [`live-tour-session.md`](examples/live-tour-session.md)) — see `process.md` and `principles.md`, "Offer a .tour export where it fits, never assume it"
- [ ] Validate the `.tour` export against the real CodeTour extension in VS Code — so far it's spec-and-example only, not runtime-verified the way the Copilot/Claude deploys were with the real `apm` CLI

## License

MIT — see [LICENSE](LICENSE).
