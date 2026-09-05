# guided-pr-review

A guided, component-by-component peer review process for AI coding assistants — built to replace file-by-file diff review with something that actually helps a reviewer understand *and* argue about a PR.

## The problem

File-by-file review of AI-generated (or any) implementations is exhausting and low-value: it surfaces line noise, not design decisions, and it gives no sense of how the pieces fit together before you're three files deep.

## What this does instead

- **Anchors on the PR**, not on a spec — every PR gets reviewed the same way.
- **Uses spec/plan context when it exists** (e.g. from SpecKit) to check implementation against stated intent — this is the actual differentiator over a generic AI reviewer. When no spec is found, it falls back to structural + risk-based review from the diff and codebase alone.
- **Gives a structural overview first** — what changed, how components connect, the data flow through the change — before touching any code.
- **Walks through components one at a time**, in call/dependency order, linking directly to the relevant file and line, and pointing out specifically what to look at and why.
- **Asks, rather than dictates** — surfaces its read on a design choice and a specific question, then waits. Pushback changes its assessment; it doesn't argue past you.
- **Captures the decision and the reasoning**, not just a verdict, so "why we built it this way" survives past the conversation.
- **Resumes from where it left off** — after you push a fix, it re-checks only what changed and carries forward everything already resolved, rather than restarting the review.
- **Same output shape every time** — a fixed template (see [`core/output-template.md`](core/output-template.md)) so review output is predictable and scannable regardless of what's in the diff.

## Status

Early work in progress. Core process is designed; the Copilot integration is the first one being built out.

## Repository structure

```
core/                       # platform-agnostic process definition — the source of truth
├── process.md              # the phases: input resolution, overview, component
│                            #   walkthrough, feedback capture, resumable re-review
├── output-template.md      # the fixed output skeleton
└── principles.md           # behavioral rules (propose don't dictate, ground in
                             #   artifacts, bias toward fewer/sharper points)

code-assistants/            # thin adapters — one per AI coding tool
└── copilot/                # GitHub Copilot chat mode + prompt file
    ├── .github/chatmodes/review-facilitator.chatmode.md
    ├── .github/prompts/review-pr.prompt.md
    └── README.md

examples/                   # worked example sessions, for reference
└── rate-limiter-session.md

scripts/
└── sync.sh                 # copies a code-assistants/<tool>/ adapter into a
                             #   consuming repo's .github/ (or equivalent) folder
```

Adapters are intentionally thin: they register the process with a given tool's extension mechanism and point back at `core/` rather than duplicating it. Adding support for a new AI coding assistant means adding a new folder under `code-assistants/`, not editing the process itself.

## Using it with GitHub Copilot

See [`code-assistants/copilot/README.md`](code-assistants/copilot/README.md) for install steps. In short: copy `.github/chatmodes/review-facilitator.chatmode.md` and `.github/prompts/review-pr.prompt.md` into your project's `.github/` folder, then run `/review-pr` from a Copilot Chat session on the branch or PR you want reviewed.

## Roadmap

- [ ] Finish and validate the Copilot adapter
- [ ] Claude Code adapter
- [ ] Cursor adapter
- [ ] `scripts/sync.sh` for pulling an adapter into a consuming repo

## License

MIT — see [LICENSE](LICENSE).
