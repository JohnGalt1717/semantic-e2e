# Plan: semantic-e2e skill pack

Build the skill pack. Do not implement a test runner, and do not fork any app.
The plan is the spec. A later agent executes it box by box.

Check a box only when the acceptance line under it is true.
Do not reopen a settled decision. If a file fact contradicts this plan, stop and ask.

## Settled decisions

- Package is a multi-skill repo. Install with `npx skills add JohnGalt1717/semantic-e2e`. `install` is an alias of `add`.
- Skills land in the consumer's agent skill dirs (project `.agents/skills/` by default). This repo does not vendor consumer config.
- Three kinds of thing:
  - `semantic-e2e` core skill: schema, record, replay, chain, impact, refusals. Names no tool.
  - `setup-semantic-e2e`: infer, propose, grill, apply, probe, commit. Re-runnable.
  - `semantic-e2e-head-<id>`: one skill per driver. Declares capabilities and an install note.
- Head config is consumer-owned: `.semantic-e2e/heads.yaml`, `.semantic-e2e/tooling.lock`, `.semantic-e2e/heads/<id>.yaml`, `.semantic-e2e/scripts/`.
- A script stores the contract (intent, executable preconditions, falsifiable assertions). Bindings (role, label, key, route, test id) are a cache from the last successful grounding, not the truth.
- The agent may re-ground a moved widget. It may not rewrite an assertion to match what it found. A healed path that changes the outcome is a failed test.
- A step names its head. Two heads may compose in one script (canvas head + Appium for the OS sheet, canvas head + Aspire for backend health). No silent fallback.
- Appium is a narrow head: `accept_permission` and `deny_permission` only. It does not drive in-app UI.
- Marionette is the shipped default canvas head because it is an MCP server and the app keeps its own `MarionetteConfiguration`. `flutter-skill` is a second canvas head with the same verbs, not the schema.
- Setup never asks a fact the repo already answered. It grills one decision at a time, with a recommended answer, and waits.
- Probe is a live handshake. A file on disk is not a pass. Do not commit a config whose required probe failed.
- Commit allowlist: `.semantic-e2e/**`, `skills-lock.json`, and the MCP snippet file the accepted plan named. App source only if the user accepted that patch. No secrets, HAR, cookies, or traces.
- Impact on day one is harvested globs plus route and symbol tokens, one hop through `requires` / `provides`. No call graph.
- Worked example is a stock Flutter login, not a product screen.

## Target layout

```text
skills/semantic-e2e/SKILL.md
skills/semantic-e2e/references/script.schema.json
skills/semantic-e2e/references/refusals.md
skills/semantic-e2e/scripts/validate.py
skills/setup-semantic-e2e/SKILL.md
skills/setup-semantic-e2e/references/grill.md
skills/setup-semantic-e2e/references/probe.md
skills/semantic-e2e-head-marionette/SKILL.md
skills/semantic-e2e-head-marionette/references/capabilities.yaml
skills/semantic-e2e-head-marionette/references/install.md
skills/semantic-e2e-head-flutter-skill/SKILL.md
skills/semantic-e2e-head-flutter-skill/references/capabilities.yaml
skills/semantic-e2e-head-flutter-skill/references/install.md
skills/semantic-e2e-head-appium-permissions/SKILL.md
skills/semantic-e2e-head-appium-permissions/references/capabilities.yaml
skills/semantic-e2e-head-appium-permissions/references/install.md
skills/semantic-e2e-head-aspire/SKILL.md
skills/semantic-e2e-head-aspire/references/capabilities.yaml
skills/semantic-e2e-head-aspire/references/install.md
skills/semantic-e2e-head-playwright/SKILL.md
skills/semantic-e2e-head-playwright/references/capabilities.yaml
skills/semantic-e2e-head-playwright/references/install.md
examples/stock-flutter-login/README.md
examples/stock-flutter-login/scripts/login.yaml
```

Each `SKILL.md` has YAML frontmatter with `name` (must match the directory name) and `description` (when to load it). Body stays short. Long material goes in `references/` and is loaded on demand.

## Phase 0 — schema and core skill

- [ ] Write `skills/semantic-e2e/references/script.schema.json` for `semantic-e2e/v1`.
  Required: `schema`, `id`, `heads`, `intent`, `steps`.
  A step requires `id`, `head`, `intent`, `action`, `assertion`.
  Optional: `requires`, `provides`, `fixtures`, `impacts`, `target.binding`, `inputs`, `recovery`, `fallback`.
  `inputs[].from` must be an env var reference. Reject inline secret-looking values.
  Acceptance: `validate.py` rejects a step with no assertion, an inline password, and a step whose `head` is not listed on the script.
- [ ] Write `skills/semantic-e2e/scripts/validate.py`. Stdlib only. Reads a YAML script and the schema. Exit 0 on valid, 1 on invalid, message names the field.
  Acceptance: the two example failures above exit 1; `examples/stock-flutter-login/scripts/login.yaml` exits 0.
- [ ] Write `skills/semantic-e2e/SKILL.md`.
  Teach record, replay, chain, impact, and the refusals in `references/refusals.md`.
  Record: explore the described intent with the bound heads, crystallize YAML, replay three times with retries off, then ask the user to accept.
  Replay: re-resolve each step against live semantics, act, check the original assertion. Update a binding only after the assertion holds. Two failed re-grounds and the script is stale, not green.
  Chain: DAG of `requires` / `provides`. `on_fail: run` may run a named script. If the predicate cannot be evaluated, abort. A mutating script must declare fixture ownership and teardown or it cannot be chained.
  Impact: at crystallization, store globs of files opened and route or symbol tokens of length >= 4. On a diff, mark `required` (path glob), `suspected` (token), `unaffected`. One hop through provides/requires.
  Acceptance: an agent given only this skill can explain the refusal rules without naming a driver.

### Refusals the core skill must state

- Green with no quoted evidence.
- Skipping or rewriting an assertion to make a run pass.
- Installing or patching before the user accepts the draft.
- Secrets, tokens, PII, raw HAR, cookies, or production URLs in a script.
- Chaining a mutating script with no teardown.
- An impact claim for a script with no impact map.
- Binding a step to a head that is not installed.
- An action outside the head's declared capabilities.
- Silent fallback.
- Hardcoding an app name, port, route, key style, MCP path, biometric policy, or default canvas driver.

## Phase 1 — shipped heads

Each head skill is instructions plus two reference files. No daemon in this repo.

`capabilities.yaml` shape:

```yaml
id: marionette
scope: [open, tap, type, scroll, assert_visible, screenshot, read_logs]
not: [accept_permission, call_api]
install_skill: semantic-e2e-head-marionette
```

- [ ] Marionette head. Install note: `marionette_flutter` in the app (debug-only binding), `dart pub global activate marionette_mcp`, MCP stdio entry, VM service URI. Probe: MCP starts, tool list contains `get_interactive_elements` and `tap`, attach succeeds or returns a precise reason.
- [ ] flutter-skill head. Install note: Dart package, not the empty npm wrapper. Probe: MCP starts, tool list contains inspect and tap, debug VM answers.
- [ ] Appium permissions head. Scope is only `accept_permission` and `deny_permission`. Install note: Appium server, platform capability, accept vs deny. Probe: `GET /status` ready, and a session can see a permission surface or reports the device missing. A missing device is a recorded gap, not a green, if the user defers it.
- [ ] Aspire head. Scope is backend health, traces, and named log fields. Probe: `aspire doctor` or `list_resources` returns; declared resource is healthy or absent.
- [ ] Playwright head. Scope is web open, click, type, assert title or role. Probe: browser opens the configured base URL and returns a title.
  Acceptance: setup can read every head's `capabilities.yaml` without loading that head's SKILL.md body. A third party can copy the marionette folder, rename it `semantic-e2e-head-<id>`, and be discovered by the `semantic-e2e-head-*` prefix. No core change.

## Phase 2 — setup skill

Write `skills/setup-semantic-e2e/SKILL.md` plus `references/grill.md` and `references/probe.md`.

Loop, first run and every later run:

1. Inventory, no questions. Scan `pubspec.yaml`, `*.csproj`, `package.json`, AppHost, AndroidManifest, Info.plist, `.mcp.json`, `.vscode/mcp.json`, agent MCP config, `skills-lock.json`, debug entrypoints, permission strings, and `.semantic-e2e/` if present. Classify each fact known, inferred, or missing.
2. Propose. Head set, already wired, should add, contradictions. Do not apply.
3. Grill the frontier only. One decision per turn. Shape: a `?` title, the body, then a `->` recommended answer grounded in the inventory. Wait. Stop when every material branch is confirmed or explicitly deferred. A deferral is written as a gap.
4. Apply only the accepted plan. Install missing head skills via `npx skills add`. Write MCP snippets from that head's `references/install.md`. Write `.semantic-e2e/`. Do not edit app source unless a question accepted that patch.
5. Probe each enabled head using the pass condition in Phase 1. Red returns to one grill question. Do not commit.
6. Commit the allowlist. Message on first accept: `chore(semantic-e2e): accept head config`. Body lists head ids and probe pass/fail. Later runs: `chore(semantic-e2e): update head config`, and only if the diff is non-empty and the new probe is green.

Inference rules, no product names:

- `marionette_flutter` in pubspec -> recommend Marionette. Read widget types into head config if `MarionetteConfiguration` exists.
- Another canvas driver package and no Marionette -> recommend that head.
- Neither canvas driver -> recommend the shipped Marionette head and say the app binding is missing.
- Permission strings or a permission package, and no canvas head can see the system sheet -> recommend Appium, then ask accept versus deny.
- AppHost or a health endpoint -> recommend Aspire.
- A web host -> recommend Playwright.

Re-run is a diff. Unchanged heads that already probed are not re-asked. A new dependency is a proposed add. A removed dependency is a destructive question. Removing a head requires an explicit confirm. Clean tree and green probes: report clean, do not commit.

Acceptance: on a fixture repo with `marionette_flutter` and an Android `CAMERA` permission, the proposal names Marionette and Appium and does not ask which debug entry exists if `lib/main.dev.dart` is present. A failed probe does not produce a commit.

## Phase 3 — stock example

- [ ] `examples/stock-flutter-login/` documents a two-screen Flutter app (login, home) with one `ValueKey` on the submit control. Do not vendor a binary.
- [ ] `scripts/login.yaml` is a valid v1 script: head `marionette`, env-var inputs, assertion `surface_visible: home` with `evidence_required: true`, impact globs for the example lib.
  Acceptance: `validate.py` exits 0 on that file. README says this example is the day-one path, not a product flow.

## Phase 4 — docs and install check

- [ ] README install block matches `npx skills add JohnGalt1717/semantic-e2e` and lists the skills a stranger should pick: core, setup, one canvas head.
- [ ] `npx skills add JohnGalt1717/semantic-e2e --list` shows the seven skills (core, setup, five heads).
  Acceptance: that list command exits 0. Do not publish to a skills index until this passes.

## Explicitly later

- GitHub Action that comments required and suspected scripts on a PR.
- Dart analyzer and Roslyn call-graph impact. Day-one globs stay.
- Angular, React, and Blazor as further Playwright capability profiles, each its own head skill.
- Auto-repair of stale scripts.

## Done

A stranger can `npx skills add` this repo, run setup against a Flutter app that has no product-specific wiring, get a probed `.semantic-e2e/` commit, and record one login script. A second party can add a head skill without a core release. Fulcrum and Fieldist are consumers of that install, not modules in this repo.
