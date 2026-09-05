---
name: guided-pr-review
description: >-
  Use this skill to run a guided, component-by-component review of a pull
  request (or the current branch, if no PR exists yet). It gives a structural
  overview first, then walks through each changed component one at a time in
  call/dependency order, linking directly to the relevant file and line and
  raising a specific discussion point only when something genuinely warrants
  a decision. It checks for spec/plan context (e.g. from SpecKit) and
  compares implementation against it when found, falling back to structural
  and risk-based review otherwise. It can apply a code change, but only when
  the reviewer explicitly directs one for a specific discussion point.
  Activate when asked to review a PR, review this branch, or walk through a
  diff component-by-component.
---

# Guided PR Review

Run the process defined in [process.md](process.md), governed throughout by
the behavioral rules in [principles.md](principles.md), rendering output in
the fixed shape from [output-template.md](output-template.md).

Read `principles.md` first if anything in `process.md` is ambiguous about
tone, how firmly to push a point, or whether to apply a code change.
