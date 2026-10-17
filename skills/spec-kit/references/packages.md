# Packages: Extensions, Presets, Workflows, Steps, Bundles

Official reference (authoritative for commands, flags, and semantics): [extensions](https://github.github.io/spec-kit/reference/extensions.html) · [presets](https://github.github.io/spec-kit/reference/presets.html) · [workflows](https://github.github.io/spec-kit/reference/workflows.html) · [bundles](https://github.github.io/spec-kit/reference/bundles.html) · [artifacts](https://github.github.io/spec-kit/reference/artifacts.html). The summaries below are shortcuts — check the docs when behavior matters.

## Component types

| Type | Purpose | Adds behavior? | Location |
| --- | --- | --- | --- |
| Extension | New commands, hooks, templates, scripts | Yes (new names) | `.specify/extensions/<id>/` |
| Preset | Override/adapt existing templates and commands | No (modifies) | `.specify/presets/<id>/` |
| Workflow | YAML orchestration of commands/gates/shell | Runs commands | `.specify/workflows/<id>/` |
| Step | Custom step type (Python package) for workflows | Executes code | `.specify/workflows/steps/<id>/` |
| Bundle | Versioned, pinned composition of the above | No (distribution) | Source: `.specify/bundles/<id>/`; records in `.specify/bundle-records.json` |

Template resolution order (first match wins): `.specify/templates/overrides/` → presets (lowest priority number first) → extensions → core `.specify/templates/`.

Extension hooks are surfaced before/after core commands (`before_specify`, `after_tasks`, `after_implement`, ...). Hooks live in `.specify/extensions.yml` (committed; enablement, not install state).

## Catalogs and trust

- Built-in stack (implicit — no config file, and `init` writes none): official catalog (install-allowed) + community catalog (discovery-only). **The community catalog needs no registration**: it is already active for `search`. Never flip it to install-allowed — that is the vetting boundary, not a missing setting.
- Install a vetted discovery-only entry directly: `specify extension info <id>` prints a candidate archive URL; then `specify extension add <id> --from <url>` (default-deny trust confirmation; non-interactive init: `--trust-extension-urls`). Curating your own install-allowed catalog is the other supported route.
- **A project `<kind>-catalogs.yml` replaces the built-in stack, it does not extend it**: after `specify extension catalog add <url> --name ours --install-allowed`, only `ours` is active — official and community catalogs are gone until you list them again. Bundles differ: `bundle-catalogs.yml` merges, the built-in stack always stays as fallback.
- Manage per kind: `specify extension|preset|workflow|bundle catalog list|add|remove`, and `specify workflow step catalog ...` for steps. `bundle catalog add` uses `--policy install-allowed|discovery-only`; the other kinds use `--install-allowed/--no-install-allowed`.
- Config files (committed project intent): `.specify/extension-catalogs.yml`, `.specify/preset-catalogs.yml`, `.specify/workflow-catalogs.yml`, `.specify/step-catalogs.yml`, `.specify/bundle-catalogs.yml`. User scope: `~/.specify/...`; `specify init --force` leaves them untouched.
- Env overrides replace every catalog with one hard-coded install-allowed entry — they bypass the vetting boundary: `SPECKIT_CATALOG_URL`, `SPECKIT_PRESET_CATALOG_URL`, `SPECKIT_WORKFLOW_CATALOG_URL`, `SPECKIT_STEP_CATALOG_URL`.
- Lower priority number = higher precedence; the first catalog that lists an id wins. `discovery-only` sources appear in search/info but refuse installs.

Review catalogs supplied by an unfamiliar project (`catalog list`) before installing from it — a project providing config is not evidence anything was vetted.

## Extensions

```bash
specify extension search [query] [--tag T] [--author A] [--verified]
specify extension info <id> [--versions] [--json]
specify extension add <id> [--version V] [--from URL] [--dev DIR] [--force] [--priority N]
specify extension list [--json] [--available | --all]
specify extension update [<id>]        # version-driven; no id = all
specify extension enable|disable <id>
specify extension set-priority <id> <N>   # lower wins conflicts
specify extension remove <id> [--keep-config] [--force]
```

- `--version` uses the winning install-allowed catalog only; it never falls back to a lower-priority source.
- Config layers (highest wins): extension defaults < `<ext>-config.yml` (commit) < `<ext>-config.local.yml` or `local-config.yml` (machine-local, ignored) < `SPECKIT_<EXT>_*` env vars.
- New extension config: copy the shipped template (name varies, e.g. `<ext>-config.template.yml` or `config-template.yml`) to `.specify/extensions/<ext>/<ext>-config.yml`.
- Bundled `agent-context` (`specify extension add agent-context`, opt-in) maintains the agent instruction file's `<!-- SPECKIT START -->`…`<!-- SPECKIT END -->` block and nothing else; the file is picked per integration (`AGENTS.md` for pi/opencode/codex/generic, `CLAUDE.md` for claude, `.github/copilot-instructions.md` for copilot), overridable via `context_file`/`context_files`.

## Presets

```bash
specify preset search [query]
specify preset info <id> | specify preset list
specify preset add <id> [--version V] [--from URL] [--dev DIR] [--priority N]
specify preset resolve <template-name>      # which layer wins for a name
specify preset set-priority <id> <N>        # lower wins; ties break alphabetically
specify preset enable|disable|update|remove ...
```

- Presets override, they do not merge, unless the entry uses a composing strategy (`prepend`, `append`, `wrap`). `replace` wins outright at the lowest priority number.
- `requires.extensions` in `preset.yml` warns (not fails) on missing/stale/disabled/version-unsatisfied extensions.

## Workflows and steps

```bash
specify workflow search | info <id> [--versions] | list
specify workflow add <id|url|path> [--version V] [--from URL] [--dev PATH]
specify workflow run <id|file.yml> -i key=value [--json]
specify workflow status [<run_id>] [--json]
specify workflow resume <run_id> [-i key=value]
specify workflow enable|disable|update|remove ...
specify workflow step add <id> [--dev DIR | --from URL | --version V] [--force]
specify workflow step list|info|remove ...
```

- Run state lives in `.specify/workflows/runs/<run_id>/`. States: `created`, `running`, `completed`, `paused` (gate), `failed`, `aborted`. `--json` prints one machine-readable object.
- Local file runs work outside a project; state then goes under the cwd's `.specify/`.
- Project overlays customize installed workflows without editing them: `specify workflow overlay add|list|enable|disable|set-priority|remove`, stored in `.specify/workflows/overlays/<workflow-id>/` (committed). Inspect composition with `specify workflow resolve <id>`.
- Step packages contain executable Python; only install from sources you trust. They load on `workflow add|run|resume`.

## Bundles

```bash
specify bundle search [query] [--offline]
specify bundle info <id> [--versions]        # fully expanded, pinned component set == install plan
specify bundle install <id|path|artifact.zip> [--version V] [--refresh] [--offline] [--integration KEY]
specify bundle init [<id>]                   # ensure project initialized, optionally install bundle
specify bundle list [--json] | update [<id>|--all] | remove <id>
specify bundle validate [--path DIR|bundle.yml]
specify bundle build [--path DIR] [--output OUTDIR]
specify bundle catalog list|add|remove ...
```

Semantics to respect:

- The source may be a catalog id, a `.zip` artifact, a bundle **directory** (containing `bundle.yml`), or a `bundle.yml` file. Local sources resolve without consulting the catalog stack; the components are still installed through the per-kind machinery, so their catalog policy applies.
- `install` initializes the project on demand; it is idempotent (matching pins are skipped) and records provenance.
- All component pins must match exactly (compatible newer *packaged patch* releases warn and are used). One version per component id across all bundles.
- Changing a recorded bundle's version/pins requires `--refresh` (local sources: pass the revised path with `--refresh`).
- `build` needs a `README.md` next to `bundle.yml` and packs that directory; `--output` decides where the artifact lands (default: next to the manifest).
- `remove` uninstalls only components the bundle contributed; shared or independently installed components stay.
- On failed install: no record is written, components from that run are removed best-effort.

First-party examples: `specify bundle install bugfix` (bug extension + bugfix workflow), `specify bundle install assess` (idea assessment).

## Verify after install

- `specify <kind> list` (add `--json` for machine-readable output).
- `specify bundle list --json` shows required vs contributed vs installed versions.
- If a new command is missing in the agent, restart the agent, then check `specify extension list` and the agent's command/skill directory.
- For conflicts: `specify preset resolve <name>` (templates), `specify workflow resolve <id>` (overlays).
