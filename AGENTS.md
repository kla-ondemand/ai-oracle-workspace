# Repository Guidelines

## Project Structure & Module Organization

This is a local multi-repo workspace tracked as a thin git superproject; treat each directory under `agents/` and `projects/` as independently versioned (agents are nested repos ignored by the superproject; projects are git submodules). Top-level reference material lives in `knowledge/`, dated journal notes live in `notes/`, and reusable skill definitions live in `skills/`. Agent scaffolding is in `agents/_template/`; create new agents with `agents/_template/scripts/incarnate.sh <name>` rather than copying files manually. Third-party study clones are under `projects/` and should be considered read-only unless a task explicitly targets them.

## Build, Test, and Development Commands

There is no workspace-level build or test command. Run commands from the relevant nested repo after checking its `README.md`, `AGENTS.md`, `CLAUDE.md`, `package.json`, `pyproject.toml`, or `Makefile`.

Examples:

```bash
cd projects/openclaw && pnpm test
cd projects/hermes-agent && pytest
cd agents/_template && ./scripts/check-agent-neutrality.sh
```

For agent documentation changes, always run the agent's `./scripts/check-agent-neutrality.sh` from that agent root.

## Coding Style & Naming Conventions

Follow the style of the nested repo you are editing. For workspace documents, use concise Markdown with descriptive headings. Name durable knowledge files in kebab case, for example `knowledge/projects/openclaw.md`. Name notes as `YYYY-MM-DD-title.md` and add them to `notes/README.md` under the correct year. Agent behavior belongs in `AGENTS.md`; in generated agents, tool-specific instruction files such as `CLAUDE.md` may be symlinks to it.

## Testing Guidelines

Tests are project-specific. TypeScript projects commonly use package scripts and colocated `*.test.ts` files; Python projects may use `pytest`. Prefer the smallest relevant test first, then run the repo's broader lint, type-check, or build gate before finishing. If no test command is documented, inspect package metadata and report what was verified.

## Commit & Pull Request Guidelines

The root workspace has no commit history. In nested repos, match local history: common examples include `fix(cli): ...`, `docs: ...`, `Feat: ...`, and concise imperative messages such as `Preserve symlinks in incarnate.sh`. Keep commits scoped to one repo and one concern. For pull requests, include the purpose, changed areas, verification commands, and linked issues; add screenshots for UI-facing changes.

## Security & Configuration Tips

Do not commit secrets, local credentials, or generated private state. Do not push or rewrite third-party project repos unless explicitly instructed. When working inside an agent repo, use task-specific worktrees rather than switching branches in place.
