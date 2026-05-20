# Workspace

Local multi-repo workspace for studying agents and projects, taking notes, and accumulating durable knowledge. Tracked as a thin git superproject: `projects/` are git submodules, `agents/` are independent nested repos ignored by the superproject.

For agent / Claude Code orientation, read [`CLAUDE.md`](CLAUDE.md) first.

## Layout

| Path | Purpose | Index |
|------|---------|-------|
| [`agents/`](agents/) | Personal agents. Each is its own git repo, built from [`agents/_template/`](agents/_template/). | — |
| [`knowledge/`](knowledge/) | Durable, topic-organized reference material. Kebab-case filenames, no date prefix. | [`knowledge/README.md`](knowledge/README.md) |
| [`notes/`](notes/) | Dated journal-style notes. Filename: `YYYY-MM-DD-{title}.md`. | [`notes/README.md`](notes/README.md) |
| [`projects/`](projects/) | Third-party repos cloned for study. Treat as read-only unless explicitly told otherwise. | — |
| [`skills/`](skills/) | Shared skill library — portable, **not auto-loaded**. Copy/symlink into a `.claude/skills/` to activate. | [`skills/README.md`](skills/README.md) |

## Skills

Two locations, different roles:

- **`.claude/skills/`** — auto-loaded by Claude Code when invoked at this workspace root. Currently: `/learning` (read/write learning summaries; writes to `notes/` or `knowledge/`).
- **`skills/`** — portable library shared across agents/surfaces. Agents adopt a shared skill by symlinking from their own `.claude/skills/`. See [`skills/README.md`](skills/README.md) for the adoption command.

## Conventions

- **Agents** follow the protocol in [`agents/_template/AGENTS.md`](agents/_template/AGENTS.md). Don't copy the template by hand — use `agents/_template/scripts/incarnate.sh <name>`. Run `./scripts/check-agent-neutrality.sh` after editing agent docs.
- **Projects** are not forks. Don't commit upstream unless the user explicitly asks.
- **Notes** are dated journal entries — one topic per file, kebab-case title. Append to `notes/README.md` grouped by year.
- **Knowledge** entries are durable; edit them over time rather than dating them. Group by topic in `knowledge/README.md`.

## Current Contents

- **Agents:** `agent-data-science`, `template`
- **Projects:** `openclaude`, `openclaw`, `maw-js`, `oracle-framework`, `hermes-agent`, `thClaws` — see [`knowledge/projects/`](knowledge/projects/) for per-project orientation summaries.
