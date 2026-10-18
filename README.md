# Spec Kit Skill

> **Spec Kit, handled.** Teach your coding agent to initialize projects, manage extensions, presets, workflows, and bundles, author new packages, and keep `.specify/` clean.

![Agent Skills](https://img.shields.io/badge/Agent%20Skills-compatible-blue)
![Validated](https://img.shields.io/badge/agentskills--ref-valid-brightgreen)
![Spec Kit](https://img.shields.io/badge/Spec%20Kit-1.0.x-6f42c1)
[![skills.sh](https://skills.sh/b/markuswondrak/spec-kit-skill)](https://skills.sh/markuswondrak/spec-kit-skill)

An [Agent Skill](https://agentskills.io) for [GitHub Spec Kit](https://github.com/github/spec-kit). It gives any compatible coding agent the operational knowledge that the docs spread across a dozen pages:

- **Initialize** — `specify init`, agent integrations, CLI lifecycle, non-interactive setups.
- **Install** — extensions, presets, workflows, steps, bundles: search, trust, pin, configure, update.
- **Author** — manifests, composition strategies, dev loops, `validate` / `build` / publish.
- **Version control** — commit intent, reconstruct installations, prove it with the fresh-clone test.

## Quick start

Install with the cross-agent [skills](https://github.com/vercel-labs/skills) CLI (opencode, Claude Code, Codex, Cursor, ...):

```bash
npx skills add markuswondrak/spec-kit-skill
```

Claude Code users can add the plugin marketplace instead:

```bash
claude plugin marketplace add markuswondrak/spec-kit-skill
claude plugin install spec-kit@spec-kit-skill
```

Or copy manually (keep the directory name `spec-kit`):

```bash
# opencode
cp -r skills/spec-kit ~/.config/opencode/skills/

# Claude Code
cp -r skills/spec-kit ~/.claude/skills/

# generic Agent Skills location
cp -r skills/spec-kit ~/.agents/skills/
```

Or keep it project-scoped: `cp -r skills/spec-kit .opencode/skills/` (or `.claude/skills/`).

## Try it

Short prompts that hit the ground running:

```text
What's the spec-kit status here?
Init spec-kit here for opencode, non-interactive; ask me about lean/bugfix and whether to apply the strict .specify VCS policy or keep Spec Kit's default, then proceed.
Show me specify bundle info bugfix, then install it pinned.
Search the spec-kit catalogs for issue-tracking extensions, summarize trust and versions, install the best fit.
Scaffold a spec-kit preset that enforces our review checklist, test it with --dev.
Create a spec-kit workflow that runs specify -> plan with a gate in between and test it locally.
Bootstrap a fresh clone using its bundle declaration; report missing pins.
Declare our installed spec-kit stack as a project-owned bundle at .specify/bundles/<project> and verify a clean restore.
```

## What's inside

| Path | Contents |
| --- | --- |
| `skills/spec-kit/SKILL.md` | Routing table, ground rules, status report, three core flows, prompt templates, pitfalls |
| `skills/spec-kit/references/init.md` | CLI install/upgrade, `specify init` flags, integrations, env vars |
| `skills/spec-kit/references/packages.md` | Extension/preset/workflow/step/bundle commands, catalogs, trust model |
| `skills/spec-kit/references/authoring.md` | Package ownership rules, manifests, strategies, dev loops, publishing |
| `skills/spec-kit/references/vcs-policy.md` | Policy choice (strict vs. Spec Kit default), intent vs. installed artifacts, project-owned bundle source, fresh-clone test |
| `skills/spec-kit/assets/specify-gitignore.txt` | Drop-in rules for the strict `.specify` version-control policy |
| `skills/spec-kit/assets/agent-commands-gitignore.txt` | Root-`.gitignore` rules for agent command/skill directories outside `.specify/` |
| `.claude-plugin/` | Claude Code plugin + marketplace manifests (auto-discovers `skills/`) |

The agent loads `SKILL.md` when relevant and pulls in references only as needed.

## Design principles

- **Packages, not project files.** Catalogs declare, bundles pin, installs reconstruct.
- **Commit intent, reconstruct installations.** Registries and installed copies never get committed next to the components they describe.
- **Let the user choose the policy.** Strict is one option, not the default; the agent never applies it silently.
- **Pin everything.** Versions, preset priorities, composition strategies — deliberately.
- **Vet, then install.** Community catalogs are discovery-only; review before `--from`.
- **Don't restate the docs.** Commands and manifests live in the [official docs](https://github.github.io/spec-kit/); the skill adds the operational layer and links out instead of duplicating.

## Verified

- Frontmatter and structure pass the official validator: `uvx --from skills-ref agentskills validate ./skills/spec-kit`
- Claude Code manifests pass `claude plugin validate .`
- Installable via `npx skills add markuswondrak/spec-kit-skill` and `claude plugin marketplace add markuswondrak/spec-kit-skill`
- Commands and flags checked against `specify` 1.0.13
- Agent command directories outside `.specify/` are classified as installed artifacts: a project with `pi` + `opencode`, and a skills-style `claude` project, reach a clean `git status` right after `specify init --force` / `bundle install`; extension commands (`speckit.bug.*`) are covered too, and hand-written prompts plus third-party skills stay visible
- The gitignore policy is tested against a real project: installed artifacts ignored, project config tracked, `local-config.yml` still ignored
- `specify init --here --force` rewrite behavior checked on 1.0.13: managed `.specify/.gitignore` and integration manifests are regenerated — the VCS policy documents the placement that keeps restores clean
- A project-owned bundle source at `.specify/bundles/<id>/` works end to end on 1.0.13: `validate`, `install <dir>`, `--refresh`, and `build --output` succeed, `init --force` leaves the directory untouched, the strict rules do not match `bundle.yml` (only the built `*.zip`), and `git status` is clean after install
- Catalog stack semantics checked on 1.0.13: a project `<kind>-catalogs.yml` **replaces** the built-in stack (official + community disappear until re-listed), `bundle-catalogs.yml` merges instead, and the community catalog needs no registration — `init` writes no catalog file and `catalog list` reports "Using built-in default catalog stack"
- Fresh-clone restore verified end to end (real `git clone` of a strict-policy project + `specify bundle install .specify/bundles/<id>`): components and workflow come back and `git status` is clean — the asset now also ignores `.specify/.workflow-install.lock`, the 0-byte flock the CLI leaves at the `.specify/` root after every workflow install
- VCS policy choice checked on 1.0.13: the flow asks strict vs. Spec Kit's default (never in CI), the policy is detected from the working tree afterwards instead of re-asked, and placement follows the bundle answer — `.specify/.gitignore` with a bundle, root `.gitignore` with `.specify/` prefixes without one

## Compatibility

Works with any agent that supports the Agent Skills format — opencode, Claude Code, and friends. Spec Kit itself works with 30+ agent integrations; this skill is agent-agnostic and describes the CLI.

## License

MIT
