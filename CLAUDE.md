# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Workspace Is

This is a **multi-repo workspace** that groups together independently-versioned agents and projects under one tree for local exploration and study. The workspace itself is a thin git superproject — its only tracked content is documentation, indexes, and submodule pointers. Build, test, and lint commands live inside each sub-repo; there is no workspace-level toolchain.

`projects/` are git submodules; `agents/*/` are nested-but-**gitignored** in the superproject (only `agents/README.md` is tracked). This means edits inside an agent repo will not appear in the workspace's `git status` — switch into the agent directory to see its own status. Fresh clones need `git clone --recurse-submodules` (or `git submodule update --init` afterward) to populate `projects/`.

## Layout

| Path | Contents |
|------|----------|
| `agents/` | One subdirectory per agent. Each agent is its own independent git repo (not a submodule) following the template in `agents/_template/`. Working trees are gitignored at the workspace level. |
| `knowledge/` | Durable, topic-organized reference material. Kebab-case filenames, no date prefix. Currently holds per-project orientation summaries under `knowledge/projects/`. Index: `knowledge/README.md`. |
| `notes/` | Dated journal-style notes. File format: `YYYY-MM-DD-{title}.md`. Index: `notes/README.md`. Distinct from `knowledge/` — notes are point-in-time entries; knowledge entries are edited over time. |
| `projects/` | Cloned third-party repos for study, tracked as git submodules pinned to specific commits. Treat as read-only unless explicitly told otherwise; do **not** commit changes upstream without explicit instruction. |
| `skills/` | **Shared skill library** — portable `SKILL.md`-format skills. Not auto-loaded. Agents/surfaces adopt them by symlinking into their own `.claude/skills/`. Distinct from `.claude/skills/` (which *is* auto-loaded for this workspace; currently provides `/learning`). See `skills/README.md` for adoption procedure. |

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

## Notes & Knowledge

- **`notes/`** — dated journal-style entries (session diaries, one-off observations). Append new files to `notes/README.md` grouped by year. **Manual channel only** — do not route automated skill output here.
- **`knowledge/`** — durable topic-organized reference. Kebab-case filenames, no date prefix. Append to `knowledge/README.md` grouped by topic.
- **`/learning` skill output ALWAYS goes to `knowledge/`** in this workspace — this overrides the skill's default routing (which would otherwise send single-session learnings to `notes/`). Rationale: this workspace treats `/learning` output as durable reference material that grows over time, not session-bound diary entries. Even on the first pass of a topic, write the entry to `knowledge/{topic-kebab}.md` in the knowledge template (one-sentence definition + topic sections), then update or expand it on later passes rather than creating a new dated note.
- For an orientation summary on any `projects/<name>/` study clone, check `knowledge/projects/<name>.md` first before re-reading the upstream repo.
