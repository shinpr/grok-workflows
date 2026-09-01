# grok-workflows

Run [claude-code-workflows](https://github.com/shinpr/claude-code-workflows) (CCW) design, implementation, diagnosis, and review recipes on Grok without maintaining a Grok-specific fork.

This marketplace provides upstream CCW workflow packs together with `claude-to-grok`, a small adapter for Grok's tool and agent names. CCW continues to own the recipes, approval gates, review loops, and quality checks.

## Quick start

Install the adapter and one workflow pack. `dev-workflows-fullstack` is the usual choice when a repository contains both backend and frontend code.

```bash
grok plugin marketplace add shinpr/grok-workflows
grok plugin install claude-to-grok --trust
grok plugin install dev-workflows-fullstack --trust
```

Confirm the installation, then open a new Grok session so its slash commands refresh:

```bash
grok plugin list
grok inspect
```

Start a workflow:

```text
/recipe-implement "Add rate limiting to the public API"
```

`marketplace add` registers the catalog; the two `plugin install` commands install the adapter and workflow pack.

## Choose a workflow pack

Install `claude-to-grok` plus one of these packs:

| Workflow pack | Use it for | Example entry point |
|---------------|------------|---------------------|
| `dev-workflows-fullstack` | Backend and React/TypeScript work in the same repository | `/recipe-fullstack-implement` |
| `dev-workflows` | Backend, API, CLI, and general development | `/recipe-implement` |
| `dev-workflows-frontend` | React and TypeScript frontend work | `/recipe-front-design` |

The packs also include focused workflows such as `/recipe-diagnose`, `/recipe-review`, and `/recipe-reverse-engineer` where applicable.

## How it works

The external plugin entries in [`.grok-plugin/marketplace.json`](.grok-plugin/marketplace.json) track the default branch of the upstream CCW repository. Workflow source and release versions remain owned by CCW.

When a recipe runs, `claude-to-grok` maps the Claude Code runtime names used by CCW to Grok equivalents. It covers:

- Claude Code tool names such as `Agent`, `Bash`, `Read`, and `Edit`
- plugin-qualified specialist names such as `dev-workflows-fullstack:task-executor`
- subagent waiting and runtime-failure reporting

The adapter does not rewrite CCW recipes or remove their user approvals, document gates, review cycles, or quality checks. `recipe-*` workflows remain user-invoked slash commands.

## Installation details

### Installed but inactive

Fresh installs are normally enabled. If `grok plugin list` shows a plugin as disabled, enable it explicitly:

```bash
grok plugin enable claude-to-grok
grok plugin enable dev-workflows-fullstack
```

Replace `dev-workflows-fullstack` with the workflow pack you installed.

### Marketplace UI

After adding the marketplace source, you can install the same plugins from Grok's Marketplace tab. Use `grok plugin list` and `grok inspect` to confirm the result.

## Troubleshooting

### Multiple marketplaces provide the same workflow pack

If another marketplace already provides `dev-workflows-fullstack`, Grok may reject the ambiguous plugin name. List the registered sources, remove the duplicate source, then install again from this marketplace:

```bash
grok plugin marketplace list
grok plugin marketplace remove <source>
```

Removing a marketplace may also uninstall plugins available only from that source.

### Subagents are missing after installation

Run `grok inspect` and confirm that `claude-to-grok`, `recipe-*`, and the selected pack's agents are present. CCW `v0.22.3` and earlier used agent frontmatter that Grok does not load; reinstall through this marketplace to use the current upstream source.

## For maintainers

### Local verification

Point Grok at a local clone when testing unpublished changes:

```bash
grok plugin marketplace add /absolute/path/to/grok-workflows
grok plugin install claude-to-grok --trust
grok plugin install dev-workflows-fullstack --trust
```

Validate the local adapter manifest with:

```bash
grok plugin validate plugins/claude-to-grok
```

### Repository layout

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

### Updating CCW compatibility

1. Review upstream CCW changes when the external workflow packs update.
2. Compare upstream CCW tool and agent names with the maps in `plugins/claude-to-grok/skills/claude-to-grok/SKILL.md`.
3. Keep the adapter focused on runtime identifiers, subagent waiting, and runtime-failure handling.
4. Validate the adapter and check the final diff before publishing.

## License

MIT
