# Agents

One directory per agent. Each agent is its own independent git repo — **not** a submodule of this workspace. The workspace superproject explicitly ignores agent working trees via `agents/*/` in the root `.gitignore`; only this `README.md` is tracked at the workspace level.

## How agents are created

Use the scaffold script — never copy the template by hand:

```bash
agents/_template/scripts/incarnate.sh <agent-name>            # creates agents/agent-<name>/
agents/_template/scripts/incarnate.sh <agent-name> <path>     # creates at a specific path
```

`incarnate.sh` clones the template, sets up the `CLAUDE.md`/`GEMINI.md`/`CODEX.md`/`KIMI.md` symlinks to `AGENTS.md`, fills in the config manifest, and initializes a fresh git history.

After incarnating: edit `AGENTS.md` and `persona/README.md`, replace the `{{placeholders}}`, then run `./scripts/check-agent-neutrality.sh` from the new agent's root.

## Conventions

- **`AGENTS.md` is the single source of truth** for agent behavior. `CLAUDE.md`, `GEMINI.md`, `CODEX.md`, `KIMI.md` are symlinks to it (in incarnated agents — in the template repo itself, `CLAUDE.md` is standalone).
- **Two-tier file format:** `AGENTS.md` is plain Markdown; every other `.md` file uses Obsidian format (YAML frontmatter, `[[wikilinks]]`, `> [!callout]` blocks). Don't mix them.
- **Task isolation:** each task gets its own git worktree under `/tmp/<taskId>`. Do not branch-switch inside an agent repo.
- **Backend neutrality:** the template uses `{{camelCase}}` placeholders everywhere a model ID, tool name, or backend-specific path would otherwise be hardcoded. After editing template docs, run `./scripts/check-agent-neutrality.sh` to validate.

## Index

| Path | Role | Status |
|------|------|--------|
| [`_template/`](_template/) | Base profile + `incarnate.sh` scaffold. Underscore prefix marks it as the template, not a working agent. | Scaffold |
| [`agent-coder/`](agent-coder/) | Best-practice coding agent (BeBe — good-natured, composed). Reading/Writing/Testing/Safety/Verification standing rules baked into `AGENTS.md`. Model: `claude-opus-4-7` primary, `claude-sonnet-4-6` fallback. | Active |
| [`agent-data-science/`](agent-data-science/) | Data scientist / analyst (BaBa — direct, formal). Notebook + report deliverables, calibrated uncertainty, reproducibility gates, structured analysis report shape. Model: `claude-sonnet-4-6` primary. | Active |
| [`agent-qa/`](agent-qa/) | Code quality reviewer (BoBo — soft-spoken). A careful second pair of eyes on diffs; reports findings, does not author product code. Models not yet pinned in `config.manifest.json`. | Incarnated, awaiting model config |
| [`agent-reviewer/`](agent-reviewer/) | Code / design / docs reviewer + content creator (Bobie — warm, cheerful). Read-mostly for reviews; write-mostly when authoring READMEs, release notes, tutorials, PR descriptions. Model: `claude-sonnet-4-6` primary, `claude-haiku-4-5` fallback. | Active |

## See Also

- [`_template/AGENTS.md`](_template/AGENTS.md) — base profile (memory, session lifecycle, git/worktree, verification gates).
- [`_template/conventions.md`](_template/conventions.md) — naming rules, placeholder table, YAML schemas, validation checklist.
- [`_template/trust-model.md`](_template/trust-model.md) — T0–T3 trust pipeline for cloned agent security.
- [`_template/neutrality.md`](_template/neutrality.md) — why and how to keep profiles backend-neutral.
- [`../CLAUDE.md`](../CLAUDE.md) — workspace-level orientation, including how agents fit into the superproject.
