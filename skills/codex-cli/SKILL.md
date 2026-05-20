---
name: codex-cli
description: Drive OpenAI's Codex CLI — prompt, sandbox modes, approval policy, config, custom agents (AGENTS.md), MCP, and common workflows.
when_to_use: User mentions "codex", "openai codex", "codex cli", "gpt agent", "openai coding agent", or asks to run / configure / migrate to OpenAI Codex. Also triggers when invoking `codex` in shell or editing `~/.codex/config.toml` / `AGENTS.md` for a Codex target.
user-invocable: true
---

# Codex CLI Skill

OpenAI's open-source agentic coding CLI — the GPT-side counterpart to Claude Code, Gemini CLI / Antigravity CLI, and Kimi Coder. Reads code, edits files, runs commands, and orchestrates multi-step tasks from a terminal.

This skill captures the knobs an agent needs to **drive** Codex CLI: invocation patterns, approval/sandbox policy, config file layout, and the AGENTS.md convention it shares with this workspace.

## When To Use

- The user asks to run, configure, install, or migrate to OpenAI Codex CLI.
- The user wants to invoke `codex` as a sub-tool inside a larger workflow (e.g. a reviewer agent that runs Codex to author a patch, or a multi-LLM evaluation).
- The user is comparing Codex CLI to Claude Code / Antigravity CLI / Kimi Coder and needs concrete differences.
- A project's `AGENTS.md` needs adjusting because it will be read by Codex CLI alongside other tools.

Do **not** invoke this skill for:
- General OpenAI API usage outside the CLI (use a generic API skill).
- ChatGPT desktop / web (not the same product surface).

## Invocation

```bash
# Interactive session in current directory:
codex

# One-shot prompt, exit after the answer:
codex exec "explain what the auth module does"

# Pipe a file as context:
codex exec - < tasks/refactor.md

# Resume the last session:
codex resume
```

Codex respects the current working directory as the project root; `AGENTS.md` is loaded if present (see [AGENTS.md](#agentsmd-shared-convention)).

## Approval & Sandbox Policy

Codex CLI runs commands and writes files under an **approval policy** that gates side-effecting actions. Modes (set via flag, config, or interactive toggle):

| Mode | Behaviour | When to use |
|------|-----------|-------------|
| `suggest` | Codex proposes the action; user confirms each one. | Default for unfamiliar repos / production work. |
| `auto-edit` | Auto-applies edits inside the workspace; still confirms shell commands. | Trusted refactors with low blast radius. |
| `full-auto` | Auto-applies edits **and** runs shell commands in a sandbox. | Throwaway spikes, sandboxed CI, ephemeral worktrees. |

**Sandbox isolation** is the second axis — separate from approval mode:

- macOS: uses `sandbox-exec` profiles to restrict filesystem and network.
- Linux: uses Landlock + seccomp where available; falls back to namespaced subprocess otherwise.
- Network access defaults **off** inside `full-auto` — explicitly enable per-session if the task needs internet.

**Rule of thumb:** never run `full-auto` on a directory whose state you cannot afford to lose. Pair it with a git worktree (`worktree` skill) for safe experimentation.

## Config

User-level config lives at `~/.codex/config.toml`:

```toml
# Model selection
model = "gpt-5-codex"           # or "gpt-4.1", "o4-mini", etc.

# Default approval policy
approval_policy = "suggest"     # suggest | auto-edit | full-auto

# Sandbox
sandbox.network = false
sandbox.writable_paths = ["/tmp", "${workspace}"]

# MCP servers (see MCP section)
[[mcp_servers]]
name = "github"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
```

Project-level overrides go in `.codex/config.toml` at the repo root. Project config wins over user config for matching keys.

## AGENTS.md (shared convention)

Codex CLI reads `AGENTS.md` from the project root on every run and treats it as standing instructions. This is the same convention used by Claude Code (via `CLAUDE.md` symlink), Gemini / Antigravity CLI (`GEMINI.md`), and Kimi Coder (`KIMI.md`).

**For this workspace:** every agent under `agents/<name>/` already follows the pattern — `AGENTS.md` is the source of truth, with backend-specific filenames symlinked to it. Codex CLI picks up the agent's `AGENTS.md` automatically when run from inside an agent directory.

When editing `AGENTS.md` to support Codex:
- Keep it backend-neutral. Use `{{placeholder}}` for model IDs, tool names, paths.
- Codex respects the same headings/sections Claude does; no special markers needed.
- After editing, run the agent's `./scripts/check-agent-neutrality.sh` to verify no provider lock-in slipped in.

## MCP (Model Context Protocol)

Codex CLI is MCP-compliant — it can connect to any MCP server (GitHub, Slack, filesystem, etc.) the same way Claude Code and Cursor do. Register servers in `~/.codex/config.toml` under `[[mcp_servers]]` (see config example above) or per-project in `.codex/config.toml`.

Use the **same MCP server binaries** the user has installed for Claude Code / other tools — there's no Codex-specific MCP build needed.

## Custom Agents (subagents)

Project-scoped sub-agents live in `.codex/agents/<name>.md`. Each file is a SKILL-like instruction sheet describing what the sub-agent does, its tools, and its constraints. Codex selects them when their description matches the current task.

This is the closest analogue to Claude Code's `Agent` tool / `subagent_type` parameter, but **file-based** rather than runtime-registered.

## Common Workflows

### One-shot patch with review

```bash
# In an isolated worktree:
git worktree add /tmp/refactor-auth -b refactor/auth
cd /tmp/refactor-auth
codex exec "extract the token-refresh logic from auth.ts into a separate module; keep the public API unchanged"
# Review the diff, then:
git diff
```

### Multi-LLM comparison

Run the same task through Codex and Claude Code, compare outputs:

```bash
codex exec "$(cat task.md)" > /tmp/codex-output.md
claude -p "$(cat task.md)" > /tmp/claude-output.md
diff /tmp/codex-output.md /tmp/claude-output.md
```

Useful for the workspace's `agent-reviewer` to spot-check whether the chosen LLM is materially different on a given task.

### Migrating from Claude Code

- `CLAUDE.md` → also read by Codex if symlinked to `AGENTS.md` (which is this workspace's convention already).
- `.claude/settings.json` permissions → not portable. Codex uses the approval-policy + sandbox model instead.
- `.claude/skills/<name>/SKILL.md` → not auto-loaded by Codex. Either convert to `.codex/agents/<name>.md`, or reference the skill content from `AGENTS.md` directly.
- MCP servers — fully portable; same server binaries work in both.

## Gotchas

- **`full-auto` + no worktree = lost work.** Always pair it with a throwaway worktree. The sandbox prevents network/filesystem damage outside the workspace, but it cannot undo destructive git operations inside it.
- **Model selection affects approval prompts.** Smaller models (e.g. `o4-mini`) may propose more low-confidence actions; consider raising the approval threshold or using `suggest` mode with cheaper models.
- **`AGENTS.md` is read every run.** Long `AGENTS.md` files inflate every prompt. Keep them tight; push lengthy context into linked files Codex can read on demand.
- **Codex does not auto-install MCP servers** — they must be installed and on PATH before being referenced in config.
- **Project `.codex/config.toml` is NOT gitignored by default.** Decide explicitly whether to commit it (team-shared config) or gitignore it (per-developer).

## See Also

- [[../../knowledge/google-antigravity]] — Google's competing agentic CLI; same AGENTS.md convention, different surface model.
- `agents/_template/AGENTS.md` — workspace's neutral agent protocol; already Codex-compatible.
- `agents/_template/neutrality.md` — why backend-neutral placeholders matter when multiple agentic CLIs read the same `AGENTS.md`.
