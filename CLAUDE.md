# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Workspace Is

This is a **multi-repo workspace** that groups together independently-versioned agents and projects under one tree for local exploration and study. The workspace itself is a thin git superproject — its only tracked content is documentation, indexes, and submodule pointers. Build, test, and lint commands live inside each sub-repo; there is no workspace-level toolchain.

## Layout

| Path | Contents |
|------|----------|
| `agents/` | One subdirectory per agent. Each agent is its own git repo following the template in `agents/_template/`. |
| `knowledge/` | Shared notes / reference material (currently empty). |
| `notes/` | Personal notes. File format: `YYYY-MM-DD-{title}.md`. `notes/README.md` is the index. |
| `projects/` | Cloned third-party repos for study. Each is its own git repo — treat as read-only unless explicitly told otherwise. |
| `skills/` | **Shared skill library** — portable `SKILL.md`-format skills. Not auto-loaded. Agents/surfaces adopt them by symlinking into their own `.claude/skills/`. Distinct from `.claude/skills/` (which *is* auto-loaded for this workspace). See `skills/README.md` for adoption procedure. |

## Working Inside an Agent (`agents/<name>/`)

Each agent repo follows a fixed protocol — **read the agent's own `AGENTS.md` (or `CLAUDE.md` symlinked to it) before doing anything else**. Key conventions inherited from the template:

- `AGENTS.md` is the single source of truth for agent behavior. `CLAUDE.md`, `GEMINI.md`, `CODEX.md`, `KIMI.md` are symlinks to it (except in the template repo itself, where `CLAUDE.md` is a standalone file describing the template).
- Two-tier file format: `AGENTS.md` is plain Markdown; all other `.md` files use Obsidian format (YAML frontmatter, `[[wikilinks]]`, `> [!callout]` blocks). Don't mix them.
- After editing any agent documentation, run `./scripts/check-agent-neutrality.sh` from that agent's root.
- New agents are created by running `agents/_template/scripts/incarnate.sh <agent-name>` — never copy the template by hand.
- Each task gets its own git worktree under `/tmp/<taskId>`; do not branch-switch inside an agent repo.

## Working Inside a Project (`projects/<name>/`)

Each `projects/` subdirectory is an independent upstream repo (cloned from GitHub). Build/test/lint commands are project-specific — check that repo's own `README.md`, `CLAUDE.md`, or `package.json`/`Makefile`/etc.

Do not commit changes to project repos unless the user explicitly asks — these are study clones, not forks.

## Notes Index

`notes/README.md` indexes files matching `YYYY-MM-DD-{title}.md`. When adding a note, append it to the index grouped by year.
