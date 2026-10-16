# Spec Kit Skill

> **Spec Kit, handled.** Teach your coding agent to initialize projects, manage extensions, presets, workflows, and bundles, author new packages, and keep `.specify/` clean.

![Agent Skills](https://img.shields.io/badge/Agent%20Skills-compatible-blue)
![Validated](https://img.shields.io/badge/agentskills--ref-valid-brightgreen)
![Spec Kit](https://img.shields.io/badge/Spec%20Kit-1.0.x-6f42c1)

An [Agent Skill](https://agentskills.io) for [GitHub Spec Kit](https://github.com/github/spec-kit). It gives any compatible coding agent the operational knowledge that the docs spread across a dozen pages:

- **Initialize** — `specify init`, agent integrations, CLI lifecycle, non-interactive setups.
- **Install** — extensions, presets, workflows, steps, bundles: search, trust, pin, configure, update.
- **Author** — manifests, composition strategies, dev loops, `validate` / `build` / publish.
- **Version control** — commit intent, reconstruct installations, prove it with the fresh-clone test.

## Quick start

Copy the skill into your agent's skills directory (keep the directory name `spec-kit`):

```bash
# opencode
cp -r spec-kit ~/.config/opencode/skills/

# Claude Code
cp -r spec-kit ~/.claude/skills/

# generic Agent Skills location
cp -r spec-kit ~/.agents/skills/
```

Or keep it project-scoped: `cp -r spec-kit .opencode/skills/` (or `.claude/skills/`).

## Try it

Short prompts that hit the ground running:

```text
What's the spec-kit status here?
Init spec-kit here for opencode, non-interactive; ask me about lean/bugfix first, then apply the strict .specify VCS policy.
Show me specify bundle info bugfix, then install it pinned.
Search the spec-kit catalogs for issue-tracking extensions, summarize trust and versions, install the best fit.
Scaffold a spec-kit preset that enforces our review checklist, test it with --dev.
Create a spec-kit workflow that runs specify -> plan with a gate in between and test it locally.
Bootstrap a fresh clone using its bundle declaration; report missing pins.
```

## What's inside

| Path | Contents |
| --- | --- |
| `SKILL.md` | Routing table, ground rules, status report, three core flows, prompt templates, pitfalls |
| `references/init.md` | CLI install/upgrade, `specify init` flags, integrations, env vars |
| `references/packages.md` | Extension/preset/workflow/step/bundle commands, catalogs, trust model |
| `references/authoring.md` | Package ownership rules, manifests, strategies, dev loops, publishing |
| `references/vcs-policy.md` | Intent vs. installed artifacts, strict `.gitignore` policy, fresh-clone test |
| `assets/specify-gitignore.txt` | Drop-in rules for the strict `.specify` version-control policy |

The agent loads `SKILL.md` when relevant and pulls in references only as needed.

## Design principles

- **Packages, not project files.** Catalogs declare, bundles pin, installs reconstruct.
- **Commit intent, reconstruct installations.** Registries and installed copies never get committed next to the components they describe.
- **Pin everything.** Versions, preset priorities, composition strategies — deliberately.
- **Vet, then install.** Community catalogs are discovery-only; review before `--from`.
- **Don't restate the docs.** Commands and manifests live in the [official docs](https://github.github.io/spec-kit/); the skill adds the operational layer and links out instead of duplicating.

## Verified

- Frontmatter and structure pass the official validator: `uvx --from skills-ref agentskills validate ./spec-kit`
- Commands and flags checked against `specify` 1.0.13
- The gitignore policy is tested against a real project: installed artifacts ignored, project config tracked, `local-config.yml` still ignored
- `specify init --here --force` rewrite behavior checked on 1.0.13: managed `.specify/.gitignore` and integration manifests are regenerated — the VCS policy documents the placement that keeps restores clean

## Compatibility

Works with any agent that supports the Agent Skills format — opencode, Claude Code, and friends. Spec Kit itself works with 30+ agent integrations; this skill is agent-agnostic and describes the CLI.

## License

MIT
