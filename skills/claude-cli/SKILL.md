---
name: claude-cli
description: Drive Anthropic's Claude Code CLI (`claude`) — interactive default, `-p`/`--print` for one-shots, six-value `--permission-mode`, granular `--allowedTools`/`--disallowedTools`, MCP and plugin subcommands, worktree+tmux session model, `--bare` for clean-room runs, and the AGENTS.md/CLAUDE.md convention shared with Codex / Antigravity / Kimi.
when_to_use: User mentions "claude code", "claude cli", "anthropic cli", or asks to run / configure / migrate to Claude Code. Also triggers when invoking `claude` in shell, editing `~/.claude/settings.json` / `.claude/`, configuring an MCP server for Claude, or comparing Claude Code's CLI surface to Codex / Antigravity / Kimi.
user-invocable: true
---

# Claude Code CLI Skill

Anthropic's agentic coding CLI — the Anthropic-side counterpart to Codex, Antigravity CLI, and Kimi Coder. Reads code, edits files, runs commands, and orchestrates multi-step tasks from a terminal.

This skill is **verified against `claude --help` output as of 2026-05-21 (CLI v2.1.146)**, not extrapolated. Claude Code's surface is the largest of the four agentic CLIs in this workspace — this skill captures the parts an agent actually needs to **drive** it from the outside, not the in-session UX.

## When To Use

- The user asks to run, configure, install, or migrate to Claude Code.
- The user wants to invoke `claude` as a sub-tool inside a larger workflow (e.g. a reviewer agent that runs `claude -p` to author a patch, scripted batch runs, or multi-LLM evaluation).
- The user is comparing Claude Code to Codex CLI / Antigravity CLI / Kimi Coder and needs concrete differences.
- The user is editing `~/.claude/settings.json`, `.claude/settings.local.json`, or configuring permissions, plugins, MCP servers, or agents for Claude.
- A project's `AGENTS.md` / `CLAUDE.md` needs adjusting because it will be read by `claude`.

Do **not** invoke this skill for:
- In-session UX questions ("how do I use Plan Mode?", "what does TaskCreate do?"). Those live in Claude's interactive docs; this skill covers the **outside-the-session** driving surface.
- General Anthropic API usage outside the CLI (use a generic Claude API skill).
- Claude.ai web / desktop chat (not the same product surface).

## Invocation (verified surface)

```bash
# Interactive session in current directory:
claude
claude "let's refactor the auth module"     # interactive with initial prompt

# One-shot non-interactive (the universal scripting form):
claude -p "explain what the auth module does"
claude --print "explain what the auth module does"
echo "explain it" | claude -p

# Continue / resume / fork:
claude -c                                   # continue most recent in current dir
claude --continue
claude -r                                   # picker UI
claude -r <session-uuid>                    # specific
claude -r --fork-session                    # branch on resume
claude --from-pr 1234                       # resume session linked to a PR

# Worktree + tmux session model:
claude -w                                   # create a worktree for this session
claude -w refactor-auth                     # named worktree
claude -w --tmux                            # also create a tmux session (iTerm2 panes if available)
claude -w --tmux=classic                    # force classic tmux even with iTerm2

# Bring extra directories into the session:
claude --add-dir ../shared-lib --add-dir ../proto-defs

# Pin model / effort / name:
claude --model opus                         # alias
claude --model claude-sonnet-4-6            # full name
claude --effort high                        # low | medium | high | xhigh | max
claude -n "auth refactor"                   # display name in prompt box + /resume picker

# IDE integration:
claude --ide                                # auto-connect to a single available IDE
```

### Subcommands (verified)

```bash
claude auth login|logout|status     # account management
claude doctor                       # health check (auto-updater, config, native build)
claude install [stable|latest|<v>]  # install the native build
claude update                       # check + install update
claude mcp <action>                 # see "MCP" below
claude plugin <action>              # see "Plugins" below
claude agents [options]             # manage background agents (dispatch from agent view)
claude auto-mode                    # inspect the auto-mode classifier
claude project purge [path]         # delete all Claude Code state for a project
claude setup-token                  # long-lived auth token (requires subscription)
claude ultrareview [PR# | base]     # cloud-hosted multi-agent code review of the current branch
```

> [!note] `ultrareview` is user-triggered and billed
> An agent driving `claude` should not call `claude ultrareview` autonomously. It's a paid cloud workflow the user invokes via `/ultrareview` — flag this if a workflow tries to launch it programmatically.

## Permission Model

Claude Code's permission model is **per-mode + per-tool**, with six distinct mode values plus granular allow/deny lists.

### `--permission-mode <mode>`

| Mode | Behaviour |
|------|-----------|
| `default` | Prompt for tool actions per the agent's normal flow. |
| `acceptEdits` | Auto-approve file edits; still prompt for other tools. |
| `auto` | Hands-off automation per the auto-mode classifier (`claude auto-mode` shows config). |
| `plan` | Plan-only mode — produces a plan via `ExitPlanMode` without executing. |
| `dontAsk` | Don't prompt — denial-by-silence on anything not explicitly allowed. |
| `bypassPermissions` | Skip all permission checks. |

### Granular tool control (orthogonal to mode)

```bash
claude --allowedTools "Bash(git *) Edit Read"      # comma- or space-separated
claude --disallowedTools "Bash(rm *) Write"        # denylist
claude --tools "Bash,Edit,Read"                    # restrict to a subset
claude --tools ""                                  # disable ALL tools
claude --tools default                             # all tools (reset)
```

The `Bash(git *)` syntax is a pattern allowlist — Bash commands matching `git *` pass without prompting; others fall back to mode behaviour.

### The nuclear options

- `--dangerously-skip-permissions` — bypass all permission checks. Recommended only for sandboxes with no internet access.
- `--allow-dangerously-skip-permissions` — make the bypass mode *available* in the session without defaulting to it. More conservative: the user/agent must still opt in mid-session.

> [!warning] `--dangerously-skip-permissions` + no isolation = lost work
> Pair it with `-w/--worktree` so the session runs in an isolated git worktree. The flag is intended for externally-sandboxed environments (containers, VMs); never run it on your primary working tree.

> [!note] Check the machine's actual defaults before trusting the tier descriptions
> The mode values above are the **flag-level enum**. A user's `~/.claude/settings.json` can pre-set `skipDangerousModePermissionPrompt: true` (or similar), changing what a bare `claude` invocation actually prompts for. Verify before automating destructive actions:
>
> ```bash
> claude auth status              # check identity + tier
> claude doctor                   # auto-updater + config health
> cat ~/.claude/settings.json     # user-level overrides
> cat .claude/settings.json       # project-level (if present)
> cat .claude/settings.local.json # project-local-only (gitignored)
> ```

## `-p / --print` Mode

`-p` enables non-interactive scripting. **Many flags only work with `-p`** — they're silently ignored elsewhere:

| Flag | Purpose |
|------|---------|
| `--output-format <text\|json\|stream-json>` | Default text. `stream-json` emits NDJSON. |
| `--input-format <text\|stream-json>` | Default text. `stream-json` lets you feed real-time chunks. |
| `--json-schema <schema>` | JSON Schema for structured output validation. |
| `--max-budget-usd <amount>` | Hard dollar cap for this run. Unique to Claude among the four CLIs. |
| `--no-session-persistence` | Don't write the session to disk; cannot be resumed later. |
| `--fallback-model <model>` | Auto-fallback when the primary is overloaded. |
| `--include-partial-messages` | Stream partial chunks (needs `--output-format=stream-json`). |
| `--include-hook-events` | Include hook lifecycle events in the stream. |
| `--replay-user-messages` | Echo user messages on stdout (stream-json I/O only). |

`-p` also **skips the workspace-trust dialog** — only use it in directories you trust. Settings files that fail validation are silently ignored in this mode (no error dialog).

## Config & State

User-level state: **`~/.claude/`**

```
~/.claude/
├── settings.json              # user-level config (theme, permission defaults, etc.)
├── commands/                  # user-level custom skills (/<name>)
├── plugins/                   # installed plugins
├── projects/                  # per-project state (transcripts, file history)
├── sessions/                  # session records
├── history.jsonl              # global history log
├── file-history/              # per-file edit history
└── …
```

Project-level state: **`.claude/`** in the repo root.

```
.claude/
├── settings.json              # team-shared project config (check into git)
├── settings.local.json        # per-developer project config (gitignore this)
├── skills/<name>/SKILL.md     # project-scoped skills (auto-loaded)
└── agents/<name>.md           # project-scoped agent definitions
```

**Setting source precedence** is controlled by `--setting-sources user,project,local` (comma-separated). Pass an explicit list to override the default load order.

### Inline configuration

```bash
claude --settings ~/path/to/settings.json    # load extra settings from file
claude --settings '{"theme":"dark"}'         # …or inline JSON
claude --mcp-config ~/some/mcp.json          # extra MCP servers
claude --strict-mcp-config                   # ONLY use --mcp-config; ignore others
claude --plugin-dir ./local-plugin           # session-only plugin from dir or .zip
claude --plugin-url https://…/plugin.zip     # session-only plugin from URL
claude --agents '{"reviewer":{"description":"…","prompt":"…"}}'   # inline agent defs
```

## `--bare` Mode

A minimal mode that strips most of Claude Code's auto-magic for clean-room runs:

```bash
claude --bare -p "process this" < input.txt
```

`--bare` (sets `CLAUDE_CODE_SIMPLE=1`) skips:
- hooks
- LSP integration
- plugin sync
- attribution
- auto-memory
- background prefetches
- keychain reads (auth is strictly `ANTHROPIC_API_KEY` or `apiKeyHelper` via `--settings`)
- `CLAUDE.md` auto-discovery

Skills still resolve via `/skill-name`. Context must be provided **explicitly** via `--system-prompt`, `--append-system-prompt`, `--add-dir`, `--mcp-config`, `--settings`, `--agents`, `--plugin-dir`.

Use `--bare` for:
- Scripted batch runs where the user's local environment must not leak in.
- CI / sandbox runs where you want behaviour deterministic across machines.
- Multi-LLM comparisons where you want each side to start from the same minimal state.

## AGENTS.md / CLAUDE.md (shared convention)

Claude Code reads `CLAUDE.md` from the project root by default (and `AGENTS.md` when symlinked, which is this workspace's convention). Same shared convention as Codex CLI, Antigravity CLI, and Kimi Coder.

**For this workspace:** every agent under `agents/<name>/` uses `AGENTS.md` as the source of truth, with `CLAUDE.md`, `GEMINI.md`, `CODEX.md`, `KIMI.md` symlinked to it. `claude` picks up the agent's `CLAUDE.md` (→ `AGENTS.md`) automatically when run from inside an agent directory.

When editing for cross-CLI compatibility:
- Keep it backend-neutral. Use `{{placeholder}}` for model IDs, tool names, paths.
- After editing, run the agent's `./scripts/check-agent-neutrality.sh`.
- `--bare` skips `CLAUDE.md` auto-discovery — point at it explicitly via `--add-dir` or `--system-prompt-file` if needed.

## MCP (Model Context Protocol)

Full first-class subcommand surface:

```bash
claude mcp list                                       # list configured servers
claude mcp get <name>                                 # inspect one
claude mcp add <name> <commandOrUrl> [args...]        # add stdio or URL-based
claude mcp add --transport http <name> <url>          # HTTP transport
claude mcp add --transport http <name> <url> --header "Authorization: Bearer …"
claude mcp add -e API_KEY=xxx <name> -- npx my-mcp-server   # stdio with env vars
claude mcp add-json <name> <json>                     # add by JSON string
claude mcp add-from-claude-desktop                    # ⭐ import from Claude Desktop (Mac + WSL)
claude mcp remove <name>
claude mcp reset-project-choices                      # clear .mcp.json approvals
claude mcp serve                                      # run Claude Code AS an MCP server
```

`add-from-claude-desktop` is the migration-from-desktop shortcut — analogous to `agy plugin import claude` in Antigravity.

## Plugins

Full marketplace + lifecycle surface:

```bash
claude plugin list                              # installed plugins
claude plugin install <name>                    # install from marketplace
claude plugin install <name>@<marketplace>      # specific marketplace
claude plugin i <name>                          # alias
claude plugin uninstall <name>
claude plugin update <name>                     # restart required to apply
claude plugin enable|disable <name>
claude plugin details <name>                    # component inventory + projected token cost
claude plugin marketplace                       # manage marketplaces
claude plugin validate <path>                   # validate manifest
claude plugin tag <path>                        # create {name}--v{version} git tag
claude plugin prune                             # remove unused auto-installed deps
```

Session-only plugins (not installed, just loaded for this run):

```bash
claude --plugin-dir ./my-plugin
claude --plugin-url https://example.com/plugin.zip
```

## Background Agents (`claude agents`)

`claude agents` opens the **background-agent view** — the UI/CLI for dispatching parallel agents that run in their own sessions. Flags configure what those dispatched sessions inherit:

```bash
claude agents \
  --model opus \
  --effort high \
  --permission-mode acceptEdits \
  --mcp-config ./team-mcp.json \
  --add-dir ../shared-lib \
  --plugin-dir ./review-plugin

# Scripting form — get live sessions as JSON without a TTY:
claude agents --json | jq '.[] | select(.status=="running")'
```

This is Claude's parallel-execution primitive — analogous to Antigravity's dynamic-subagent fan-out and Codex's `mcp-server` + Cloud combo, but **scoped to background sessions** rather than implicit fan-out within one turn.

## Special Surfaces

| Flag / subcommand | Purpose |
|-------------------|---------|
| `--ide` | Auto-connect to a single available IDE (VS Code, JetBrains). |
| `--chrome` / `--no-chrome` | Toggle the Claude-in-Chrome integration. |
| `--remote-control [name]` | Start an interactive session with Remote Control enabled. |
| `--remote-control-session-name-prefix` | Prefix for auto-generated Remote Control names. |
| `--from-pr [value]` | Resume the session linked to a GitHub PR (number, URL, or picker). |
| `--file <id:path…>` | Pre-download file resources at startup. |
| `--betas <name…>` | Add beta headers (API-key users only). |
| `--exclude-dynamic-system-prompt-sections` | Move per-machine sections (cwd, env, git status) into the first user message — improves cross-user prompt-cache reuse. Default off. |
| `claude install` | Install the native build (`stable`, `latest`, or specific version). |
| `claude setup-token` | Generate a long-lived API token (requires Claude subscription). |

## Common Workflows

### One-shot patch in an isolated worktree

```bash
claude -w refactor-auth --permission-mode acceptEdits -p \
  "extract the token-refresh logic from auth.ts into a separate module; keep the public API unchanged"
# The worktree path is printed by the session; cd into it to review:
cd $(find /tmp -maxdepth 2 -name 'refactor-auth' 2>/dev/null | head -1)
git diff
```

### Clean-room scripted run (no auto-magic)

```bash
claude --bare \
  --system-prompt "You produce only TOML output." \
  --output-format json \
  --max-budget-usd 0.10 \
  -p "rewrite this YAML as TOML: $(cat config.yaml)"
```

`--bare` + explicit context + a budget cap is the safest way to invoke `claude` from inside another agent's pipeline.

### Structured output via JSON Schema

```bash
claude -p "extract the version, license, and main file from package.json: $(cat package.json)" \
  --output-format json \
  --json-schema '{
    "type":"object",
    "required":["version","license","main"],
    "properties":{
      "version":{"type":"string"},
      "license":{"type":"string"},
      "main":{"type":"string"}
    }
  }'
```

### Multi-LLM comparison

```bash
claude -p "$(cat task.md)" > /tmp/claude-output.md
codex  exec "$(cat task.md)" > /tmp/codex-output.md
agy    -p   "$(cat task.md)" > /tmp/agy-output.md
diff3 /tmp/claude-output.md /tmp/codex-output.md /tmp/agy-output.md
```

### Resuming or forking a session

```bash
claude -c                          # continue the most recent in this dir
claude -r                          # picker UI
claude -r <session-uuid>           # specific session
claude -r <uuid> --fork-session    # branch instead of overwrite
```

### Adopting MCP servers from Claude Desktop

```bash
claude mcp add-from-claude-desktop
claude mcp list
```

### Migrating to / from other agentic CLIs

| From → To | Mechanic |
|-----------|----------|
| Claude Desktop → Claude Code CLI | `claude mcp add-from-claude-desktop` |
| Claude Code → Antigravity | `agy plugin import claude` (in [[../antigravity-cli/SKILL]]) |
| Claude Code → Codex | **No automatic import.** Rewrite skills as plugins, redo MCP via `codex mcp add`. |
| Claude Code → Kimi Coder | `CLAUDE.md → KIMI.md` (symlink); MCP servers portable. |

## Gotchas

- **`-p` means `--print` in Claude and Antigravity, but `--profile` in Codex.** Easy to muscle-memory the wrong tool. `claude -p "..."` and `agy -p "..."` agree; `codex -p` does something completely different.
- **Many flags only apply to `-p` mode.** `--output-format`, `--json-schema`, `--max-budget-usd`, `--fallback-model`, `--no-session-persistence`, the stream-json I/O flags — all silently no-op in interactive mode. Check this skill's `-p` table when a flag seems to be ignored.
- **Workspace trust dialog is skipped in `-p` mode.** Only run `claude -p` in directories you trust. Settings-file validation errors are silently ignored too.
- **`--dangerously-skip-permissions` + no `-w/--worktree` = lost work.** Always pair the bypass with an isolated worktree, container, or VM.
- **`--allow-dangerously-skip-permissions` ≠ `--dangerously-skip-permissions`.** The former only makes the bypass *available* in-session without enabling it; the latter enables it from the start. Pick deliberately.
- **`--bare` is a hard reset.** It strips `CLAUDE.md` auto-discovery — provide context explicitly via `--add-dir`, `--system-prompt`, `--mcp-config`, etc., or the agent flies blind.
- **`--strict-mcp-config` ignores `.mcp.json` / settings-based MCP entirely.** Use it when you need deterministic MCP behaviour for CI / scripts; otherwise local MCP servers will silently augment the run.
- **`claude ultrareview` is billed.** An agent should not invoke it autonomously — it's a user-initiated cloud workflow.
- **`claude project purge` is destructive** (deletes transcripts, tasks, file history, config entry for a project). Confirm before running.
- **Six permission modes, not three.** Don't assume Codex's two-axis model or Antigravity's binary `--sandbox` flag — Claude has `default | acceptEdits | auto | plan | dontAsk | bypassPermissions`, each with different semantics.
- **Effort levels (`--effort`) are unique to Claude.** Codex and Antigravity don't have an explicit knob; if a workflow assumes one, it's Claude-only.

## See Also

- [[../codex-cli/SKILL]] — OpenAI Codex CLI; opposite `-p` semantics, two-axis safety model, has `codex mcp add` + Cloud.
- [[../antigravity-cli/SKILL]] — Google Antigravity CLI; `-p` matches Claude, has built-in `plugin import claude|gemini`, single `--sandbox` flag.
- [[../../knowledge/google-antigravity]] — Antigravity product context (four-surface platform).
- `agents/_template/AGENTS.md` — workspace's neutral agent protocol; already Claude-compatible via the `CLAUDE.md` → `AGENTS.md` symlink.
- `agents/_template/neutrality.md` — why backend-neutral placeholders matter when multiple agentic CLIs (`claude`, `codex`, `agy`, `kimi`) read the same `AGENTS.md`.
