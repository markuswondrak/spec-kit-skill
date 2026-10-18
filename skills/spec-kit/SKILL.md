---
name: spec-kit
description: Operate GitHub Spec Kit (spec-driven development toolkit, `specify` CLI). Use when initializing spec-kit projects, installing or managing extensions, presets, workflows, or bundles, authoring spec-kit components, applying the .specify version-control policy, or when the user mentions spec-kit, specify CLI, /speckit commands, SDD, bundles, presets, or the .specify directory.
license: MIT
metadata:
  author: Markus Wondrak
  version: "1.7"
---

# Spec Kit

Spec Kit is GitHub's spec-driven development toolkit. A project gets a `.specify/` directory (constitution, templates, scripts, components) plus agent commands (`/speckit-specify`, `/speckit-plan`, ...) that walk a feature through specify → plan → tasks → implement.

This skill covers the operational side: setup, component management, authoring, and version control. Invoke the installed SDD commands directly when asked to run them — they ship with the project's agent integration.

Canonical docs: <https://github.github.io/spec-kit/> — [quickstart](https://github.github.io/spec-kit/quickstart.html), [reference](https://github.github.io/spec-kit/reference/overview.html), [community](https://github.github.io/spec-kit/community/overview.html). The reference pages are authoritative for commands and manifests; when they differ from this skill, follow the docs and treat this file as operational shortcuts.

## Route by task

| Task | Read |
| --- | --- |
| No task given — report the current state (initialized? components? runs?) | [Default: status report](#default-status-report) |
| Initialize or upgrade a project, pick agent integration, install the CLI | [references/init.md](references/init.md) |
| Search/install/update extensions, presets, workflows, bundles; catalogs | [references/packages.md](references/packages.md) |
| Create or publish an extension, preset, workflow, step, or bundle | [references/authoring.md](references/authoring.md) |
| Decide what to commit under `.specify/`; restore a fresh clone | [references/vcs-policy.md](references/vcs-policy.md) |

## Ground rules

- Run `specify` from the project root (the directory containing `.specify/`). Component commands fail without an initialized project.
- In agent contexts the CLI always runs with explicit flags plus `--non-interactive`; questions for the user are asked in conversation, not by the CLI.
- Let the user choose the VCS policy ([step 3](#flow-bootstrap-a-project)); never pick it silently. Non-interactive/CI runs use Spec Kit's default.
- Commit intent, reconstruct installations: never commit installed component copies, registries, caches, or the agent's command/skill files outside `.specify/` ([vcs-policy](references/vcs-policy.md)).
- Declare the stack once: a project with more than one component keeps a pinned bundle source at `.specify/bundles/<id>/` and restores it with `specify bundle install .specify/bundles/<id>` — remembered per-component `add` runs are not a restore path ([vcs-policy](references/vcs-policy.md)).
- Pin versions (`--version`, bundle pins). Unpinned bundle component entries are rejected.
- The community catalog is discovery-only and needs no registration (already active for `search`). Vet an entry (`info`), then install via `--from <url>`.
- Extensions, workflows, and step packages execute code. Review untrusted sources first; shell steps run unsandboxed with your privileges.
- Bump the component version on every content change — `update` is version-driven.

## Default: status report

Invoked without a concrete task, do not init or install anything — collect the state read-only, then summarize:

1. Project: `.specify/` present? Not initialized → say so and offer the bootstrap flow. Initialized → continue.
2. CLI and integration: `specify version`, `specify self check` (drift), `specify integration status`.
3. Components: `specify extension list`, `specify preset list`, `specify workflow list`, `specify bundle list` — include versions and enabled/disabled state; compare installed vs. pinned.
4. Runs in flight: `specify workflow status --json` — call out paused runs (gate) with their `run_id` and how to resume.
5. Active feature: `.specify/feature.json` (when present).
6. Working tree and VCS policy: `git status --short`; classify the policy as strict / Spec Kit default / scaffold-restored with the probes in [vcs-policy](references/vcs-policy.md) — report it, change it only on request.

Report a short bullet list, not raw command dumps. Flag what needs action (drifted pins, stale or disabled components, dirty tree, missing commands) and point to the matching flow.

## Flow: bootstrap a project

1. Check tooling: `specify check`, `specify version --features`. Install: `uv tool install specify-cli`; upgrade: `specify self upgrade`.
2. Initialize: `specify init . --integration <agent> --non-interactive` (directory non-empty: add `--force`).
3. In an interactive session, ask before init: optional first-party extras (the `lean` preset via `--preset lean`, the `bugfix` bundle via `specify bundle install bugfix` right after init) and the VCS policy — **strict** (commit intent only) or **Spec Kit's default** (commit the installed stack; nothing hidden). Then add components at init: `--preset <id>`, repeated `--extension <name|path|url>`. More than one component → declare them once in `.specify/bundles/<id>/bundle.yml` plus a two-line README. In CI/non-interactive runs, skip the question and use the default ([vcs-policy](references/vcs-policy.md)).
4. Apply the chosen policy. Strict: rules from [assets/specify-gitignore.txt](assets/specify-gitignore.txt) in `.specify/.gitignore`, or with `.specify/` prefixes in the root `.gitignore` when there is no bundle (init may be the restore path and rewrites the managed file). Either way [assets/agent-commands-gitignore.txt](assets/agent-commands-gitignore.txt) goes in the **root** `.gitignore`, one line per installed integration. Default: commit the installed stack instead; no assets ([vcs-policy](references/vcs-policy.md)).
5. Verify: `specify integration status`, `specify extension list`. Acceptance test: fresh clone + `specify bundle install .specify/bundles/<id>` (or the scaffold `specify init --here --force` line) reaches a working state with a clean `git status`; a default-policy project is clean once the stack is committed.

## Flow: install a package

1. Discover: `specify <kind> search <query>` → `specify <kind> info <id>` (trust: `verified` vs `community`).
2. For bundles, review the expanded plan first: `specify bundle info <id>`.
3. Install pinned: `specify bundle install .specify/bundles/<id>` (project-owned stack) or `specify bundle install <id>` (catalog bundle). In a project that has a bundle, adding a component means editing the manifest and re-running with `--refresh`.
4. Configure if needed: copy the shipped config template to `<ext>-config.yml`, edit, commit.
5. Verify: `specify <kind> list`; restart the agent if new commands are missing. The VCS policy is set at init only — detect it, never re-ask.

## Flow: author a package

1. Pick the owning package type ([authoring](references/authoring.md)): extension = new commands; preset = override existing behavior; workflow = orchestration; step = new step type; bundle = pinned distribution.
2. Implement, then test locally: `specify extension add --dev <dir>`, `specify preset add --dev <dir>`, `specify workflow run ./workflow.yml`, `specify bundle install ./bundle.yml`.
3. Validate: installs validate manifests; bundles also `specify bundle validate`; presets `specify preset resolve <name>`.
4. Publish: tag a release, add a catalog entry (community submissions via issue template). Bundles: `specify bundle build` produces the distributable `.zip` first.

## Prompt templates (short)

- `Init spec-kit here for <agent>, non-interactive; ask me about lean/bugfix and whether to apply the strict .specify VCS policy or keep Spec Kit's default, then proceed.`
- `Show me specify bundle info <id>, then install it pinned.`
- `Search the spec-kit catalogs for <topic>, summarize trust and versions, install the best fit from an install-allowed source.`
- `Scaffold a spec-kit preset that <behavior>, test it with --dev, verify with specify preset resolve spec-template.`
- `Create a spec-kit workflow that runs specify -> plan with a gate in between, test it from a local YAML.`
- `Group our installed spec-kit stack into a bundle at .specify/bundles/<project>, validate it, and verify a fresh-clone restore leaves git status clean.`
- `/speckit-<cmd> is missing: diagnose and fix it.`
- `Bootstrap a fresh clone using its bundle declaration; report missing pins or uncommitted intent.`

## Pitfalls

| Symptom | Cause / fix |
| --- | --- |
| Component command missing in agent | Not installed/enabled, or the agent needs a restart; check `specify <kind> list`. |
| `Extension 'x' not found` | Not in an install-allowed catalog; may be community-only → `specify extension info x`, then `--from`. |
| Bundle install refuses a version | One version per component id: another bundle or independent install pins it. |
| Fresh clone broken | Installed copies committed without pins; fix the declaration (bundle), not the artifacts. Restore with `specify bundle install .specify/bundles/<id>` and keep that command in the agent instruction file ([vcs-policy](references/vcs-policy.md)). |
| Restore depends on remembered commands | Components were added one by one; declare them in `.specify/bundles/<id>/bundle.yml` and restore with `bundle install .specify/bundles/<id>` ([vcs-policy](references/vcs-policy.md)). |
| Official extensions stop installing after adding a catalog | A project `<kind>-catalogs.yml` replaces the built-in stack — re-list the official catalog next to yours ([packages](references/packages.md)). |
| Dirty tree or vanished rules after `specify init --force` | Init rewrites `.specify/.gitignore` and re-stamps `integrations/*.manifest.json`; keep strict rules in the root `.gitignore` and ignore the manifests ([vcs-policy](references/vcs-policy.md)). |
| `?? .pi/`, `?? .opencode/`, `?? .claude/` right after init | Agent command/skill files are installed artifacts: commit them (Spec Kit default policy) or ignore `…/speckit*` in the **root** `.gitignore`, one line per installed integration ([assets/agent-commands-gitignore.txt](assets/agent-commands-gitignore.txt)); never in `.specify/.gitignore`. |
| Strict rules present although the user did not ask | The policy is a user choice; the default is Spec Kit's own. Detect with `git check-ignore` and change only on request ([vcs-policy](references/vcs-policy.md)). |
| Stale behavior after an update | `specify self check` for CLI drift; `specify <kind> update`; the upstream version was not bumped. |
