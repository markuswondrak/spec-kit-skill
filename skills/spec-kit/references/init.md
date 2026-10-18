# Init, Integrations, and CLI Lifecycle

## Prerequisites and CLI install

- Python 3.11+, [`uv`](https://docs.astral.sh/uv/), a supported agent.
- Install from PyPI: `uv tool install specify-cli`
- Pin a release: `uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@vX.Y.Z` (keep the leading `v`).
- Full install matrix (uv, pipx, PyPI, air-gapped): [installation](https://github.github.io/spec-kit/installation.html).
- Lifecycle: `specify self check` (newer release available?), `specify self upgrade` (`--tag vX.Y.Z` to pin), `specify version --features`, `specify version --json`.

## Initialize a project

```bash
specify init <name> --integration <key> --non-interactive     # new directory
specify init . --integration <key> --non-interactive          # current directory (--here works too)
specify init . --force --integration <key> --non-interactive  # merge into a non-empty directory (rewrites managed files)
```

Key options:

| Option | Purpose |
| --- | --- |
| `--integration <key>` | Agent integration. Without it, non-interactive runs default to Copilot (`SPECKIT_INTEGRATION_DEFAULT` changes the fallback). |
| `--integration-options "<opts>"` | Integration-specific flags, e.g. `--skills`, or `--commands-dir <dir>` for `generic`. |
| `--script sh\|ps\|py` | Script flavor for the feature helper scripts. |
| `--preset <id>` | Install a preset during init (first-party: `lean`, a minimal core workflow). |
| `--extension <name\|path\|url>` | Install extension(s) during init; repeatable. URL installs in non-interactive mode need `--trust-extension-urls`. |
| `--here`, `--force` | Target the current directory / skip the non-empty-directory confirmation. |
| `--non-interactive` | Never prompt; fail instead of hanging. Required for agent harnesses. |
| `--ignore-agent-tools` | Skip checks for agent CLI tools on the PATH. |

In interactive sessions, ask the user before init: optional first-party extras (`lean` preset via `--preset lean`; `bugfix` bundle via `specify bundle install bugfix` right after init) and the VCS policy — strict or Spec Kit's default ([vcs-policy.md](vcs-policy.md) → *Choosing a policy*). In CI/agent runs, skip the question and use the default policy.

Init flags describe the first commit. A project that depends on more than one component declares them once as a project-owned bundle (`.specify/bundles/<id>/bundle.yml`, installed by path) instead of repeating `--preset`/`--extension` later ([vcs-policy.md](vcs-policy.md)).

Initialization is offline: assets come from the installed CLI package, so templates always match the CLI version.

`--force` merges into an existing checkout, but managed artifacts are rewritten: `.specify/.gitignore` is reset to the managed template (appended rules are lost; [vcs-policy.md](vcs-policy.md) shows the root-`.gitignore` placement) and `.specify/integrations/<key>.manifest.json` gets a fresh `installed_at`.

Integration keys include `claude`, `copilot`, `gemini`, `opencode`, `codex`, `generic` and more; `specify integration list` shows availability ([reference](https://github.github.io/spec-kit/reference/integrations.html)). Naming surface varies by agent:

- Command style (`/speckit.specify`) — Claude in commands mode, Copilot, Gemini, opencode (`.opencode/commands/speckit.*.md`), ...
- Skills style (`/speckit-specify`, `.claude/skills/speckit-specify/SKILL.md`) — Claude now installs skills by default; Codex/Command Code skills use `$speckit-*`; Kimi uses `/skill:speckit-*`.

The generated files in `.claude/`, `.opencode/`, etc. are authoritative for the project's agent. They are installed artifacts, not intent: ignore `…/speckit*` in the **root** `.gitignore` ([../assets/agent-commands-gitignore.txt](../assets/agent-commands-gitignore.txt)), never in `.specify/.gitignore` ([vcs-policy.md](vcs-policy.md)).

## After init

1. `specify check` — required tools and installed integrations.
2. `specify integration status` — what the project currently targets.
3. Git workflows need the git extension (not installed by default):
   `specify extension add git` (bundled; works offline). In a project that declares its stack, this is an entry in the bundle instead of a one-off add.
4. Optional first-party bundle when bugs are in scope: `specify bundle install bugfix` (bug extension + bugfix workflow, pinned). The `lean` preset is passed to init itself (`--preset lean`).
5. More than one component → declare the set once in `.specify/bundles/<id>/bundle.yml` (plus a short `README.md`) and install from that path; per-component `add` runs are for experiments, never the restore path ([vcs-policy.md](vcs-policy.md)).
6. Apply the chosen VCS policy: strict per [vcs-policy.md](vcs-policy.md), or Spec Kit's default (commit the installed stack; no assets).
7. Acceptance test on a fresh clone: `git clone ... && specify bundle install .specify/bundles/<id>` (or `bundle install <bundle-id>` for a catalog bundle) must reach a working state.

## Manage integrations after init

```bash
specify integration list                    # available + installed
specify integration install <key>           # add an integration to an existing project
specify integration switch <key>            # switch the active integration
specify integration upgrade <key>           # refresh an installed integration (diff-aware)
specify integration uninstall <key>         # remove it, preserving user-modified files
specify integration use <key>               # set default without uninstalling others
```

## Environment variables worth knowing

| Variable | Effect |
| --- | --- |
| `SPECKIT_INTEGRATION_DEFAULT` | Fallback integration for `init` when `--integration` is omitted. |
| `SPECIFY_INIT_DIR` | Target a member project from outside it (must contain `.specify/`). Honored by project-scoped CLI commands and core feature scripts. |
| `SPECIFY_FEATURE_DIRECTORY` | Override the active feature directory (takes precedence over `.specify/feature.json`). |
| `SPECIFY_FEATURE_NO_PERSIST=1` | Do not rewrite `.specify/feature.json` (concurrent agents). |
| `GH_TOKEN` / `GITHUB_TOKEN` | Needed for private GitHub-hosted catalogs and archives. |

## Verify

- `specify --version` runs and matches the intended release.
- `.specify/` exists with `templates/`, `scripts/`, `memory/constitution.md`, `integration.json`.
- The agent directory contains the expected `speckit-*` commands/skills.
- VCS policy matches the init choice and is detectable ([vcs-policy.md](vcs-policy.md) → *Choosing a policy*): strict — rules in `.specify/.gitignore` or the root `.gitignore`; default — installed copies and commands tracked.
- `git status --short` is clean right after init/restore, agent command directories included: under strict from ignoring them, under the default from committing the stack.
