# Shared Skills

Portable skill library for this workspace. Skills here are **not auto-loaded** — they're a curated pool that agents and Claude Code instances copy or symlink into their own `.claude/skills/` (or equivalent).

## How it relates to `.claude/skills/`

| Location | Auto-loaded? | Scope | Purpose |
|----------|--------------|-------|---------|
| `.claude/skills/` (workspace root) | Yes, when Claude Code runs here | Workspace | Skills the workspace itself uses (e.g. `/learning`). |
| `agents/<name>/.claude/skills/` | Yes, when that agent runs | Single agent | Per-agent skills. |
| `skills/` (this directory) | **No** | Shared library | Reusable skills available for any agent or surface to adopt. Copy or symlink into a `.claude/skills/` to activate. |

## Layout

One directory per skill, named in kebab-case. Each contains a `SKILL.md` with YAML frontmatter:

```
skills/
└── <skill-name>/
    ├── SKILL.md       # required — frontmatter + instructions
    └── ...            # optional scripts, templates, references
```

### `SKILL.md` frontmatter

```yaml
---
name: <skill-name>
description: <one-line summary used to decide relevance>
when_to_use: <user phrasing / triggers that should activate it>
user-invocable: true   # true if user can type /<skill-name> directly
---
```

Body is plain Markdown instructions for the LLM when the skill is invoked. See `agents/_template/.claude/skills/session-end/SKILL.md` for a reference example.

## Adopting a Shared Skill

From an agent or surface that wants the skill active:

```bash
# Symlink (preferred — keeps it tracking the shared copy):
ln -s ../../../../skills/<skill-name> .claude/skills/<skill-name>

# Or copy (when divergence is intended):
cp -R ../../skills/<skill-name> .claude/skills/<skill-name>
```

## Index

### Agentic CLI Tooling

- [antigravity-cli/](antigravity-cli/) — drive Google's Antigravity CLI (`agy`); Go-rewritten successor to Gemini CLI, dynamic subagents, plugin model with `import claude|gemini`, `AGENTS.md`/`GEMINI.md` convention, 2026-06-18 consumer sunset.
- [claude-cli/](claude-cli/) — drive Anthropic's Claude Code CLI (`claude`); `-p` non-interactive, six-value `--permission-mode`, granular `--allowedTools`, worktree+tmux session model, `--bare` clean-room mode, MCP/plugin/agents subcommands.
- [codex-cli/](codex-cli/) — drive OpenAI's Codex CLI; two-axis safety model (`--ask-for-approval` × `--sandbox`), `~/.codex/config.toml`, MCP and plugin subcommands, OSS provider support (`--oss`), AGENTS.md convention.
- [hermes-agent/](hermes-agent/) — drive Nous Research's Hermes Agent; programmatic one-shot via `hermes chat -q -Q`, session resume, messaging-gateway service bridging Telegram/Discord/Slack/WhatsApp/Signal/IRC/Teams/Google Chat/LINE/Weixin, cron delivery, OpenClaw migration path.
- [openclaude/](openclaude/) — drive OpenClaude (Claude Code UX with the provider swapped out); `/provider` profiles, `CLAUDE_CODE_USE_OPENAI=1` env path, `ollama launch openclaude`, per-agent `agentRouting` in `~/.openclaude.json`, optional headless gRPC server for embedding.

### Research & Knowledge Capture

- [research/](research/) — externally-sourced research with persistence to `research/{topic-kebab}.md`; web search + citation, refreshable entries via `researched_at` frontmatter.

<!--
When adding a skill, append a one-line entry under the right section. Add new sections as needed. Example categories:

## Memory & Reflection
- [learning/](learning/) — read and write learning summaries.

## Session Lifecycle
- [session-start/](session-start/) — load memory + task-log at session start.

## Agentic CLI Tooling
- [codex-cli/](codex-cli/) — drive OpenAI Codex CLI.
-->
