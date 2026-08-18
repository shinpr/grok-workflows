---
name: claude-to-grok
description: >
  Maps Claude Code tool and agent identifiers to Grok runtime equivalents
  while preserving claude-code-workflows (dev-workflows*) execution semantics.
  Use when running dev-workflows* on Grok, including recipe-* orchestration
  that needs CC→Grok translation.
---

# Claude → Grok (CCW runtime map)

## Runtime invariants

1. **Preserve CCW execution** — keep every CCW step, `[Stop]` gate, handoff contract, review loop, commit cycle, and quality gate mandatory.
2. **Translate identifiers** — replace Claude Code tool/agent names with the Grok tools in the maps; keep prompt *what*, deliverable paths, and acceptance criteria unchanged.
3. **Use the required specialist** — spawn exclusively the plugin-qualified CCW agent type named by the recipe (`dev-workflows-fullstack:<name>` or the backend/frontend prefix).
4. **Handle runtime failures** — when a mapped tool or agent cannot complete, follow the Runtime failure procedure. Map only documented equivalents.

Recipe invocation remains exclusively user-slash initiated (`disable-model-invocation` on `recipe-*`). This skill activates for translation during orchestration.

## Tool name map (Claude Code → Grok)

| Claude Code (in CCW text) | Grok tool | Notes |
|---------------------------|-----------|--------|
| `Agent` / `Task` | `spawn_subagent` | Fresh call per phase. Then wait — see **Waiting for a subagent**. Use `resume_from` only when CCW explicitly continues the same subagent session. |
| `Bash` | `run_terminal_command` | git commit, ls, verification shells |
| `Read` | `read_file` | |
| `Write` | `write` | New file or full overwrite |
| `Edit` / `MultiEdit` | `search_replace` | Prefer exact old→new edits; use `write` when full-file write is clearer |
| `Grep` | `grep` | |
| `Glob` / `LS` / `ListDir` | `list_dir` + `grep` / `run_terminal_command` as needed | Discover paths via `list_dir` or shell only when list_dir is insufficient |
| `WebSearch` | `web_search` | |
| `AskUserQuestion` | `ask_user_question` | All `[Stop]` confirmations |
| `SendMessage` (resume) | `spawn_subagent` + `resume_from` | Prefer fresh `spawn_subagent` per CCW isolation; `resume_from` only when CCW continues the same session |

Orchestrator write tools: use mapped Edit/Write tools **only inside specialists** that CCW allows to edit.

## Waiting for a subagent

CCW text says “invoke Agent and use the result”; it does not name a Grok wait API.

After `spawn_subagent`:

1. Read the returned subagent id (or task id from the tool result).
2. Call `get_command_or_subagent_output` with that id (use a positive `timeout_ms` when waiting for completion).
3. WHEN the subagent is still running or the wait times out, wait again with the same id.
4. WHEN it completes with a final payload, use that payload as the specialist result and continue the CCW flow (Stop, next specialist, etc.).
5. WHEN it fails, is cancelled, or completes without a final payload, run the Runtime failure procedure.

Treat the spawn result as pending unless it already contains a completed final payload. `spawn_subagent` often runs in the background by default.

## Subagent type map

**Canonical form:**

```text
dev-workflows-fullstack:<agent-name>
```

| CCW `subagent_type` text | Grok `subagent_type` |
|--------------------------|----------------------|
| `requirement-analyzer` | `dev-workflows-fullstack:requirement-analyzer` |
| `task-executor` | `dev-workflows-fullstack:task-executor` |
| `task-executor-frontend` | `dev-workflows-fullstack:task-executor-frontend` |
| `quality-fixer` | `dev-workflows-fullstack:quality-fixer` |
| `quality-fixer-frontend` | `dev-workflows-fullstack:quality-fixer-frontend` |
| `document-reviewer` | `dev-workflows-fullstack:document-reviewer` |
| `technical-designer` | `dev-workflows-fullstack:technical-designer` |
| `technical-designer-frontend` | `dev-workflows-fullstack:technical-designer-frontend` |
| `work-planner` | `dev-workflows-fullstack:work-planner` |
| `task-decomposer` | `dev-workflows-fullstack:task-decomposer` |
| `codebase-analyzer` | `dev-workflows-fullstack:codebase-analyzer` |
| `code-verifier` | `dev-workflows-fullstack:code-verifier` |
| `security-reviewer` | `dev-workflows-fullstack:security-reviewer` |
| `acceptance-test-generator` | `dev-workflows-fullstack:acceptance-test-generator` |
| `design-sync` | `dev-workflows-fullstack:design-sync` |
| `integration-test-reviewer` | `dev-workflows-fullstack:integration-test-reviewer` |
| `prd-creator` | `dev-workflows-fullstack:prd-creator` |
| `ui-analyzer` | `dev-workflows-fullstack:ui-analyzer` |
| `ui-spec-designer` | `dev-workflows-fullstack:ui-spec-designer` |

WHEN the installed plugin is backend-only or frontend-only, use:

- `dev-workflows:<agent-name>`
- `dev-workflows-frontend:<agent-name>`

Use the plugin-qualified subagent type. WHEN it is unavailable, inspect the registered subagent types and follow the Runtime failure procedure. Use a bare name only when inspection shows that exact bare name is registered.

## Subagent prompt construction

Keep CCW prompt rules. Always include:

1. Concrete deliverable paths with every placeholder resolved
2. Expected action
3. Scope boundary block from the recipe when the recipe requires it
4. Runtime footer WHEN the selected agent instructions or delegated prompt refer to Claude Code tool names:

```text
Runtime (Grok): Agent/Task→spawn_subagent then get_command_or_subagent_output;
Bash→run_terminal_command; Read→read_file; Edit/MultiEdit→search_replace; Write→write;
AskUserQuestion→ask_user_question.
subagent_type prefix: dev-workflows-fullstack: (or dev-workflows: / dev-workflows-frontend:).
Preserve every CCW step and gate.
```

## Orchestrator tool surface (mapped)

| CCW permitted | Grok |
|---------------|------|
| Agent | `spawn_subagent` + wait via `get_command_or_subagent_output` |
| AskUserQuestion | `ask_user_question` |
| Bash | `run_terminal_command` |
| Read | `read_file` |

Implementation Edit/Write/MultiEdit remain **specialist-only** per CCW.

## Runtime failure procedure

WHEN a mapped operation cannot complete (including: no map row, agent not registered, `spawn_subagent` failed, subagent failed/cancelled, no final payload, mapped tool missing from the current surface):

1. Record the attempted CCW obligation, mapped Grok operation, and observed result.
2. Inspect the currently available Grok tools or subagent types relevant to that operation.
3. Retry only when the result indicates a transient or still-running state.
4. Continue with a documented equivalent when the current environment exposes one that appears in this skill’s maps.
5. WHEN no executable equivalent is available, report using the format below.

### Runtime failure output

```text
## Unresolved CCW→Grok runtime failure

- CCW obligation:
- Mapped Grok operation:
- Observed result:
- Available tools or subagent types checked:
- Missing capability:
- User action or information required:
```

## Scope

This skill covers runtime identifier translation, subagent waiting, and runtime-failure reporting. CCW defines recipe bodies, workflow gates, and quality policy.
