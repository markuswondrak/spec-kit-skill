---
name: spec-kit
description: Operate GitHub Spec Kit (spec-driven development toolkit, `specify` CLI). Use when initializing spec-kit projects, installing or managing extensions, presets, workflows, or bundles, authoring spec-kit components, applying the .specify version-control policy, or when the user mentions spec-kit, specify CLI, /speckit commands, SDD, bundles, presets, or the .specify directory.
license: MIT
metadata:
  author: Markus Wondrak
  version: "1.0"
---

# Spec Kit

Spec Kit is GitHub's spec-driven development toolkit. A project gets a `.specify/` directory (constitution, templates, scripts, components) plus agent commands (`/speckit-specify`, `/speckit-plan`, ...) that walk a feature through specify → plan → tasks → implement.

This skill covers the operational side: setup, component management, authoring, and version control. Invoke the installed SDD commands directly when asked to run them — they ship with the project's agent integration.

## Route by task

| Task | Read |
| --- | --- |
| Initialize or upgrade a project, pick agent integration, install the CLI | [references/init.md](references/init.md) |
| Search/install/update extensions, presets, workflows, bundles; catalogs | [references/packages.md](references/packages.md) |
| Create or publish an extension, preset, workflow, step, or bundle | [references/authoring.md](references/authoring.md) |
| Decide what to commit under `.specify/`; restore a fresh clone | [references/vcs-policy.md](references/vcs-policy.md) |

## Ground rules

- Run `specify` from the project root (the directory containing `.specify/`). Component commands fail without an initialized project.
- In agent or CI contexts, pass explicit flags plus `--non-interactive`; never rely on interactive prompts.
- Commit intent, reconstruct installations: never commit installed component copies, registries, or caches ([vcs-policy](references/vcs-policy.md)).
- Pin versions (`--version`, bundle pins). Unpinned bundle component entries are rejected.
- The community catalog is discovery-only. Vet an entry (`info`), then install via `--from <url>`.
- Extensions, workflows, and step packages execute code. Review untrusted sources first; shell steps run unsandboxed with your privileges.
- Bump the component version on every content change — `update` is version-driven.

## Flow: bootstrap a project

1. Check tooling: `specify check`, `specify version --features`. Install: `uv tool install specify-cli`; upgrade: `specify self upgrade`.
2. Initialize: `specify init . --integration <agent> --non-interactive` (directory non-empty: add `--force`).
3. Optionally add components at init: `--preset <id>`, repeated `--extension <name|path|url>` (e.g. `--extension git`).
4. Apply the strict VCS policy: append the rules from [assets/specify-gitignore.txt](assets/specify-gitignore.txt) to `.specify/.gitignore`.
5. Verify: `specify integration status`, `specify extension list`. Acceptance test: fresh clone + `specify bundle install <bundle-id>` reaches a working state.

## Flow: install a package

1. Discover: `specify <kind> search <query>` → `specify <kind> info <id>` (trust: `verified` vs `community`).
2. For bundles, review the expanded plan first: `specify bundle info <id>`.
3. Install pinned: `specify bundle install <id>` or `specify <kind> add <id> --version <v>`.
4. Configure if needed: copy the shipped config template to `<ext>-config.yml`, edit, commit.
5. Verify: `specify <kind> list`; restart the agent if new commands are missing.

## Flow: author a package

1. Pick the owning package type ([authoring](references/authoring.md)): extension = new commands; preset = override existing behavior; workflow = orchestration; step = new step type; bundle = pinned distribution.
2. Implement, then test locally: `specify extension add --dev <dir>`, `specify preset add --dev <dir>`, `specify workflow run ./workflow.yml`, `specify bundle install ./bundle.yml`.
3. Validate: installs validate manifests; bundles also `specify bundle validate`; presets `specify preset resolve <name>`.
4. Publish: tag a release, add a catalog entry (community submissions via issue template). Bundles: `specify bundle build` produces the distributable `.zip` first.

## Prompt templates (short)

- `Init spec-kit here for <agent>, non-interactive, then apply the strict .specify VCS policy.`
- `Show me specify bundle info <id>, then install it pinned.`
- `Search the spec-kit catalogs for <topic>, summarize trust and versions, install the best fit from an install-allowed source.`
- `Scaffold a spec-kit preset that <behavior>, test it with --dev, verify with specify preset resolve spec-template.`
- `Create a spec-kit workflow that runs specify -> plan with a gate in between, test it from a local YAML.`
- `Group our installed spec-kit stack into a bundle, validate it, and verify a fresh-clone restore.`
- `/speckit-<cmd> is missing: diagnose and fix it.`
- `Bootstrap a fresh clone using its bundle declaration; report missing pins or uncommitted intent.`

## Pitfalls

| Symptom | Cause / fix |
| --- | --- |
| Component command missing in agent | Not installed/enabled, or the agent needs a restart; check `specify <kind> list`. |
| `Extension 'x' not found` | Not in an install-allowed catalog; may be community-only → `specify extension info x`, then `--from`. |
| Bundle install refuses a version | One version per component id: another bundle or independent install pins it. |
| Fresh clone broken | Installed copies committed without pins; fix the declaration (bundle), not the artifacts. |
| Stale behavior after an update | `specify self check` for CLI drift; `specify <kind> update`; the upstream version was not bumped. |
