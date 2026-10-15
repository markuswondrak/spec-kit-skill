# Authoring Spec Kit Packages

## Choose the owning package type

- New command names / hooks / capabilities → **extension**.
- Change how existing commands or templates render → **preset** (composition strategies; priorities).
- Automate a sequence (commands, gates, shell, branching) → **workflow**.
- New executable step type for workflows → **step** package.
- Ship a role/team stack of pinned components → **bundle**.

Rule of thumb: a preset registering a brand-new command is a smell; that belongs in an extension. Logic in workflow YAML beyond orchestration belongs in a preset/extension script (or its own package).

## Extensions

Layout: `extension.yml`, `commands/*.md`, optional `templates/`, `scripts/`, `<ext>-config.template.yml`, `.extensionignore`, `README.md`, `LICENSE`.

Minimal manifest:

```yaml
schema_version: "1.0"
extension:
  id: "my-ext"                 # ^[a-z0-9-]+$
  name: "My Extension"
  version: "1.0.0"             # strict SemVer X.Y.Z (no v prefix, no prerelease)
  description: "One sentence."
  author: "You"
  repository: "https://github.com/you/spec-kit-my-ext"
  license: "MIT"
requires:
  speckit_version: ">=0.1.0"
provides:
  commands:
    - name: "speckit.my-ext.hello"   # ^speckit\.[a-z0-9-]+\.[a-z0-9-]+$
      file: "commands/hello.md"
      description: "Say hello"
```

Command file: Markdown with optional frontmatter (`description`, `tools`, `scripts` mapping `sh`/`ps`). Use `$ARGUMENTS` for user input. Reference sibling commands only via `__SPECKIT_COMMAND_<NAME>__` tokens (upper case, `speckit.` dropped, dots → `_`, hyphens kept) — never hard-code `/speckit.x.y`, because the invocation syntax differs per agent.

Hooks (top-level, sibling of `provides`):

```yaml
hooks:
  after_tasks:
    - command: "speckit.my-ext.hello"
      priority: 10        # integer >= 1, lower runs first
      optional: true
      prompt: "Run hello?"
```

Events: `before|after_` + one of `specify`, `plan`, `tasks`, `implement`, `analyze`, `checklist`, `clarify`, `constitution`, `taskstoissues` (plus `bug_*` for the bundled bug extension). Provide at least one of `provides.commands`, `provides.templates`, `provides.scripts`, `hooks`, `events`. Extension templates/scripts always resolve as `replace` (no `strategy` key allowed).

`.extensionignore`: `.gitignore`-style patterns for dev files excluded from the installed copy (no negation inside excluded directories; the file itself is always excluded).

Dev loop:

```bash
specify extension add --dev ./my-ext
specify extension list && specify extension info my-ext
# edit, then reinstall:
specify extension add --dev ./my-ext --force
specify extension remove my-ext
```

Publish: tag a SemVer release; users install via `--from <archive-url>`. For catalog discovery, submit via the Extension Submission issue template (community catalog is discovery-only). **Bump the version on every content change** — `specify extension update` is version-driven.

## Presets

Start from the scaffold at `presets/scaffold/` in the spec-kit repo. Layout: `preset.yml`, `README.md`, `LICENSE`, optional `templates/`, `commands/`, `CHANGELOG.md`.

Manifest essentials:

```yaml
schema_version: "1.0"
preset:
  id: "my-preset"
  name: "My Preset"
  version: "1.0.0"
  description: "Under 200 characters."
  author: "You"
  repository: "https://github.com/you/spec-kit-preset-my-preset"
  license: "MIT"
requires:
  speckit_version: ">=0.9.0"
  extensions:
    - "companion-extension"
    - id: "other-extension"
      version: ">=1.2.0,<2"   # PEP 440 specifier
      required: false
provides:
  templates:
    - type: "template"
      name: "spec-template"
      file: "templates/spec-template.md"
      description: "Custom spec template"
      replaces: "spec-template"
      strategy: "append"        # replace (default) | prepend | append | wrap
  commands:
    - type: "command"
      name: "speckit.specify"
      file: "commands/speckit.specify.md"
      strategy: "wrap"          # use {CORE_TEMPLATE} in the wrapper file
```

Composition rules: every preset defaults to priority 10; lower wins; ties break alphabetically by id. `replace` overrides the layer below (no merge); `prepend`/`append`/`wrap` compose with it. Scripts support only `replace` and `wrap`. One preset entry may target many commands by pointing multiple entries at one file with the same strategy — prefer that over copying a preamble into every command.

Dev + publish:

```bash
specify preset add --dev ./my-preset
specify preset resolve spec-template     # confirm your layer wins
specify preset remove my-preset
```

Catalog submission requires a preset-scoped README containing a literal `specify preset add ...` command; the validation workflow checks this mechanically (README link must end in `README.md`). Submit a PR adding the entry to `presets/catalog.community.json` (entries sorted by id) plus a row in `docs/community/presets.md`.

## Workflows

```yaml
schema_version: "1.0"
workflow:
  id: "my-flow"            # lowercase, hyphens; also becomes the install dir
  name: "My Flow"
  version: "1.0.0"
  author: "You"
  description: "What it does."
requires:
  speckit_version: ">=0.9.0"
  integrations:
    any: ["claude", "copilot", "gemini", "opencode"]
inputs:
  spec:
    type: string           # string | number | boolean
    required: true
    prompt: "Describe what to build"
  scope:
    type: string
    default: "full"
    enum: ["full", "backend-only"]   # constrain values that reach shell steps
steps:
  - id: specify
    command: speckit.specify
    input:
      args: "{{ inputs.spec }}"
  - id: review
    type: gate
    message: "Review the spec before planning."
    options: [approve, reject]
    on_reject: abort       # abort (default) | skip | retry
  - id: plan
    command: speckit.plan
```

Step types: `command`, `prompt`, `shell`, `github`, `init`, `slot`, `gate`, `if`, `switch`, `while`, `do-while`, `fan-out`, `fan-in`, `workflow` (call an installed workflow with declared inputs/outputs). Expressions use `{{ inputs.x }}`, `{{ steps.<id>.output.<key> }}`, `{{ context.run_id }}`; filters: `default`, `join`, `contains`, `map`, `from_json`, `to_json`, `upper`, `lower`, `split`, `length`.

Local test loop:

```bash
specify workflow run ./workflow.yml -i spec="Build a kanban board"
specify workflow info ./workflow.yml
specify workflow add ./workflow.yml --dev    # install local package (dir with workflow.yml)
```

Customize installed workflows with project overlays instead of editing them (edits survive bundle/workflow updates). Overlay anchors are base step ids; operations: `insert_after`, `insert_before`, `replace`, `remove`; lower priority value wins.

**Shell safety:** a `shell` step runs unsandboxed with user privileges. Interpolation is raw string substitution — no quoting/escaping. Constrain any interpolated value with `enum` or keep it out of `run` fields entirely. Keep agent/prompt output out of `run`; branch with `if`/`switch` instead. Review any catalog workflow before running it.

Publish: follow `workflows/PUBLISHING.md` in the spec-kit repo (catalog entry with `url` + `sha256`; community catalog).

## Step packages

```
my-step/
├── step.yml        # step.type_key must equal the CLI <step_id>
├── __init__.py     # StepBase subclass with matching type_key
└── helpers.py      # optional
```

```bash
specify workflow step add my-step --dev ./my-step
specify workflow step add my-step --dev ./my-step --force   # replace after edits
```

Limits: max 512 entries, 32 directory levels, 50 MiB retained. Installing does not import the package, but `workflow add|run|resume` does — treat like running arbitrary code.

## Bundles

```yaml
schema_version: "1.0"
bundle:
  id: "my-bundle"
  name: "My Bundle"
  version: "1.0.0"
  role: "developer"
  description: "One sentence."
  author: "You"
  license: "MIT"
requires:
  speckit_version: ">=0.9.0"
provides:
  extensions:
    - id: "bug"
      version: "1.0.1"       # every entry must be pinned
  presets:
    - id: "my-preset"
      version: "1.0.0"
      priority: 10           # preset metadata is part of the pin
  workflows:
    - id: "bugfix"
      version: "1.0.0"
  steps:
    - id: "my-step"
      version: "1.0.0"       # steps resolve via step catalogs only
```

Rules: all primitive entries pinned; no duplicate id per kind; one installed version per component id across bundles; optional `source` names the expected winning catalog (not a URL); a bundle adds no runtime behavior of its own. Bundle-local step payloads are not installed offline — publish steps to a catalog.

Workflow:

```bash
specify bundle validate [--path ./my-bundle]       # manifest + every pin resolves
specify bundle install ./bundle.yml                # local test (no catalog needed for the manifest)
specify bundle install ./bundle.yml --refresh      # apply manifest changes to a recorded bundle
specify bundle build --path ./my-bundle --output dist/   # -> versioned .zip artifact
```

Publish: host the built `.zip` and a catalog JSON entry (`id`, `version`, `download_url`, `sha256`, `requires`, `provides`; historical releases under `releases`). If the bundle pulls components from non-default catalogs, document those catalog URLs and prove a clean install. Community submissions go through the Bundle Submission issue template.

Versioning model (policy recommendation): components you co-release with the bundle share the bundle's version; foreign components keep their own versions but stay pinned. Never mix the two rules implicitly.

## Cross-cutting

- Manifests are validated on install; keep ids lowercase-hyphenated, versions strict SemVer.
- Keep version pins honest: a stale pin describes a different environment than users install.
- Update checks are version-driven — always bump before releasing content changes.
- Do not commit installed copies or registries for authored packages either, unless the project deliberately owns the source (then it belongs outside `.specify/`).
