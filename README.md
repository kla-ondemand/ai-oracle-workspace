# Workspace

Local multi-repo workspace for studying agents and projects, taking notes, and accumulating durable knowledge. Tracked as a thin git superproject: `projects/` are git submodules, `agents/` are independent nested repos ignored by the superproject.

For agent / Claude Code orientation, read [`CLAUDE.md`](CLAUDE.md) first.

Public remote: `https://github.com/kla-ondemand/ai-oracle-workspace.git`. Fresh clones should use:

```bash
git clone --recurse-submodules https://github.com/kla-ondemand/ai-oracle-workspace.git
```

After a plain clone, populate `projects/` with `git submodule update --init`.

## Layout

| Path | Purpose | Index |
|------|---------|-------|
| [`agents/`](agents/) | Personal agents. Each working agent is its own git repo, built from [`agents/_template/`](agents/_template/); only [`agents/README.md`](agents/README.md) is tracked here. | [`agents/README.md`](agents/README.md) |
| [`knowledge/`](knowledge/) | Durable, topic-organized reference material. Kebab-case filenames, no date prefix. | [`knowledge/README.md`](knowledge/README.md) |
| [`notes/`](notes/) | Dated journal-style notes. Filename: `YYYY-MM-DD-{title}.md`. | [`notes/README.md`](notes/README.md) |
| [`projects/`](projects/) | Third-party repos cloned for study. Treat as read-only unless explicitly told otherwise. | [`projects/README.md`](projects/README.md) |
| [`research/`](research/) | Externally-sourced research artifacts — citation-backed, refreshable (staleness tracked via `researched_at` frontmatter). Written by the `/research` skill. | [`research/README.md`](research/README.md) |
| [`skills/`](skills/) | Shared skill library — portable, **not auto-loaded**. Copy/symlink into a `.claude/skills/` to activate. | [`skills/README.md`](skills/README.md) |

## Skills

Two locations, different roles:

- **`.claude/skills/`** — auto-loaded by Claude Code when invoked at this workspace root. Currently: `/learning`, with a workspace override: all learnings go to `knowledge/`, never `notes/`.
- **`skills/`** — portable library shared across agents/surfaces. Agents adopt a shared skill by symlinking from their own `.claude/skills/`. Currently: `antigravity-cli`, `claude-cli`, `codex-cli`, `hermes-agent`, `openclaude` under Agentic CLI Tooling, plus `research` under Research & Knowledge Capture. See [`skills/README.md`](skills/README.md).

## Conventions

- **Agents** follow the protocol in [`agents/_template/AGENTS.md`](agents/_template/AGENTS.md). The underscore prefix marks `_template/` as the scaffold, not a working agent. Don't copy it by hand — use `agents/_template/scripts/incarnate.sh <name>`. Run `./scripts/check-agent-neutrality.sh` after editing agent docs.
- **Projects** are not forks. Don't commit upstream unless the user explicitly asks.
- **Notes** are manual dated journal entries — one topic per file, kebab-case title. Append to `notes/README.md` grouped by year.
- **Knowledge** entries are durable; edit them over time rather than dating them. Group by topic in `knowledge/README.md`, which includes project summaries and external platform/vendor notes such as `google-antigravity.md`.

## Current Contents

- **Agents:** `_template` scaffold; active — `agent-coder` (BeBe), `agent-data-science` (BaBa), `agent-reviewer` (Bobie); incarnated but awaiting model config — `agent-qa` (BoBo). See [`agents/README.md`](agents/README.md) for the full index.
- **Projects:** `hermes-agent`, `maw-js`, `openclaude`, `openclaw`, `oracle-framework`, `thClaws` — see [`projects/README.md`](projects/README.md) for the submodule index and [`knowledge/projects/`](knowledge/projects/) for per-project orientation summaries.
