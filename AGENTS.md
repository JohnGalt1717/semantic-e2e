# semantic-e2e — agent contract

This repo is a skill pack, not an application. Do not add Fulcrum, Fieldist, or any other product names, ports, routes, or key conventions to the core skill.

## What to build

Read and implement [`.agents/plan.md`](.agents/plan.md). That file is the spec. Check a phase box only when its acceptance check passes.

## Settled, do not reopen

- Install path is `npx skills add JohnGalt1717/semantic-e2e`. `install` is an alias of `add`.
- Core skill names no driver. Heads are separate skills named `semantic-e2e-head-*`.
- Head config lives in the consumer repo at `.semantic-e2e/`, not in this pack.
- A step names its head. Missing head, missing capability, or silent fallback is a failed run.
- Setup infers from the repo, grills only unsettled decisions, probes for real, then commits the allowlist.
- No green without a quoted assertion. No secrets in scripts.

## Out of scope until the plan says otherwise

Call graphs, auto-fixing scripts, a new test runner, and a GitHub App.
