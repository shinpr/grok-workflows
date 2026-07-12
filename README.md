# grok-workflows

Grok marketplace for running [claude-code-workflows](https://github.com/shinpr/claude-code-workflows) (CCW) on Grok.

CCW is a workflow suite originally written for Claude Code. It can run on Grok on its own, but tool names, agent types, and progress-tracking duties still differ. This repository packages:

1. **`claude-to-grok`** — runtime translation skill (this repo)
2. **`dev-workflows*`** — CCW workflow plugins (external entries that pull upstream CCW)

Grok users use this marketplace as the install surface. Workflow design stays upstream in CCW.

Layout is Grok-native (`.grok-plugin` only).

## Why this exists

CCW recipes and agents refer to Claude Code concepts such as `Agent`, `TaskCreate`, `Bash`, and bare `subagent_type` names.

Grok already understands many of these concepts. Remaining differences still need a stable mapping so orchestration applies the same translations every turn. That mapping is the `claude-to-grok` skill.

CCW `recipe-*` skills stay user slash commands (`disable-model-invocation: true`). This repo does not rewrite CCW flows, workflow stop points (user approval gates), document gates, or quality cycles.

## Plugins

| Plugin | Role |
|--------|------|
| `claude-to-grok` | Runtime translation (required on Grok) |
| `dev-workflows-fullstack` | Fullstack workflows + recipes (typical choice) |
| `dev-workflows` | Backend-only workflows |
| `dev-workflows-frontend` | Frontend-only workflows |

Upstream content: [claude-code-workflows](https://github.com/shinpr/claude-code-workflows).  
External entries resolve to CCW **`v0.22.4`** (see `.grok-plugin/marketplace.json`).

## Install (Grok)

You install **two plugins by name** from this marketplace: a `dev-workflows*` pack and `claude-to-grok`.

### How Grok install works

| Step | Command | What it does |
|------|---------|----------------|
| Register marketplace | `grok plugin marketplace add …` | Adds a **catalog**. No skills yet. |
| Install plugins | `grok plugin install <name> --trust` | Downloads/activates that plugin. **Required** for each plugin you use. |
| Enable | `grok plugin enable <name>` | Only if something is installed but disabled. Fresh install is usually already enabled. |

`marketplace add` alone does nothing useful for recipes.  
`enable` is not a substitute for `install`.

After `marketplace add`, `grok plugin install` accepts the **plugin name** as listed in that catalog (verified with Grok 0.2.x). You do not pass git URLs or version strings in the install line; the marketplace entry already points at the right upstream ref.

### Steps

**1. Add this marketplace (from Git)**

```bash
grok plugin marketplace add shinpr/grok-workflows
```

Same idea as other Grok marketplaces: the source is the git repo, not a local path.

**2. Install both plugins by name**

```bash
grok plugin install claude-to-grok --trust
grok plugin install dev-workflows-fullstack --trust
```

Use `dev-workflows` or `dev-workflows-frontend` instead of fullstack when that matches your stack.

Install **both**—the adapter is required for the intended Grok experience. Upstream CCW version comes from this marketplace’s catalog (currently `v0.22.4`); you do not put a CCW version on the install line.

> If install fails because **multiple marketplaces** provide `dev-workflows-fullstack` (e.g. you already added the upstream `claude-code-workflows` marketplace on Grok), remove that source first (`grok plugin marketplace remove …` — use the source as shown in `grok plugin marketplace list`), then install again via this marketplace. That may uninstall plugins that came only from the removed source.

**3. Confirm, then open a new session**

```bash
grok plugin list
grok inspect   # claude-to-grok skill + recipe-* + dev-workflows-fullstack:* agents
```

Start a **new** Grok session so slash commands refresh. Then:

```text
/recipe-implement …
```

### If a plugin is installed but inactive

```bash
grok plugin enable claude-to-grok
grok plugin enable dev-workflows-fullstack
```

Only when `list` shows the plugin as disabled—not the normal first-time path.

### UI

After the marketplace source is added, the Marketplace tab can install the same plugins. Confirm with `grok plugin list` / `grok inspect`.

### Developing this repo before publish

Until `shinpr/grok-workflows` is on GitHub, point the marketplace at a clone for local verification only:

```bash
grok plugin marketplace add /path/to/grok-workflows
# then the same install <name> --trust commands as above
```

That path form is for maintainers/smoke tests, not the documented end-user install.

## What the adapter does

### In scope

- Translate Claude Code Agent invocations (`Agent` / `Task` tool) to `spawn_subagent`
- Translate common file and shell tools to Grok names
- Canonicalize `subagent_type` to `dev-workflows-fullstack:<name>` (or the backend/frontend plugin prefix)
- Convert progress registration duty `TaskCreate` / `TaskUpdate` → `todo_write` (session todos only; not full Task board parity)
- Report runtime failures with a structured escalation after checking the available tools or subagent types (see the mapping skill)

### Out of scope (owned by upstream CCW)

- Flow design, recipe bodies, and gate policy
- Substituting built-in agents for CCW specialists
- Thinning Large/Medium planning or quality cycles

## Layout

```text
grok-workflows/
  .grok-plugin/marketplace.json    # adapter + external CCW plugins
  plugins/
    claude-to-grok/
      .grok-plugin/plugin.json
      skills/
        claude-to-grok/
          SKILL.md
  README.md
  LICENSE
```

## Maintaining compatibility

When bumping CCW for Grok users:

1. Update external plugin `ref` values in `.grok-plugin/marketplace.json` (name-based install uses this pin).
2. Diff CCW agent/tool names against `plugins/claude-to-grok/skills/claude-to-grok/SKILL.md` tables.
3. Keep the mapping skill short — identifiers, wait behavior, and duty equivalents only.

### Troubleshooting

- **Subagents not recognized after install:** catalog pin must be **v0.22.4+**. CCW `v0.22.3` agent frontmatter uses a skills format Grok does not load; reinstall via this marketplace’s pin, not an older tag.

## Relationship to CCW

| Concern | Where |
|---------|--------|
| Workflow design, recipes, agents, quality gates | [claude-code-workflows](https://github.com/shinpr/claude-code-workflows) |
| Grok marketplace + runtime name mapping | this repository |

## License

MIT
