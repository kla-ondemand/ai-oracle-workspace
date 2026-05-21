---
name: codex-cli
description: Drive OpenAI's Codex CLI — interactive + non-interactive (`codex exec`), two-axis safety model (`--ask-for-approval` × `--sandbox`), `~/.codex/config.toml`, MCP and plugin subcommands, OSS provider support (`--oss`), and the AGENTS.md convention shared with Antigravity / Claude Code / Kimi.
when_to_use: User mentions "codex", "openai codex", "codex cli", "gpt agent", "openai coding agent", or asks to run / configure / migrate to OpenAI Codex. Also triggers when invoking `codex` in shell or editing `~/.codex/config.toml` / `AGENTS.md` for a Codex target.
user-invocable: true
---

# Codex CLI Skill

OpenAI's open-source agentic coding CLI — the GPT-side counterpart to Claude Code, Antigravity CLI, and Kimi Coder. Reads code, edits files, runs commands, and orchestrates multi-step tasks from a terminal.

This skill is **verified against `codex --help` output as of 2026-05-21**, not extrapolated. The safety model in particular is not what older docs describe — it is **two orthogonal axes**, not a tri-mode policy.

## When To Use

- The user asks to run, configure, install, or migrate to OpenAI Codex CLI.
- The user wants to invoke `codex` as a sub-tool inside a larger workflow (e.g. a reviewer agent that runs `codex exec` to author a patch, or a multi-LLM evaluation).
- The user is comparing Codex CLI to Claude Code / Antigravity CLI / Kimi Coder and needs concrete differences.
- A project's `AGENTS.md` needs adjusting because it will be read by Codex CLI alongside other tools.

Do **not** invoke this skill for:
- General OpenAI API usage outside the CLI (use a generic API skill).
- ChatGPT desktop / web (not the same product surface).

## Invocation (verified surface)

```bash
# Interactive session in current directory:
codex
codex "let's refactor the auth module"     # interactive with initial prompt

# One-shot non-interactive:
codex exec "explain what the auth module does"
codex e    "explain what the auth module does"     # alias

# Read prompt from stdin:
codex exec - < tasks/refactor.md
cat tasks/refactor.md | codex exec        # equivalent (stdin appended as <stdin> block)

# Resume previous interactive session (picker by default):
codex resume                              # show session picker
codex resume --last                       # continue the most recent
codex resume <session-uuid>               # specific UUID
codex resume <thread-name>                # named thread

# Fork an existing session into a new branch:
codex fork --last
codex fork <session-uuid>

# Apply a diff produced by Codex (Codex Cloud workflow):
codex apply <task-id>
codex a    <task-id>                      # alias

# Sandbox an arbitrary shell command (not just Codex):
codex sandbox macos -- npm test            # Seatbelt on macOS
codex sandbox linux -- pytest              # bubblewrap/Landlock on Linux

# Run a non-interactive code review:
codex review

# Health check:
codex doctor
```

Other top-level subcommands: `login`, `logout`, `mcp`, `plugin`, `mcp-server`, `app-server` (exp), `remote-control` (exp), `app` (launches Codex desktop app), `cloud` (exp, Codex Cloud), `update`, `completion`, `features`, `debug`.

## Approval & Sandbox — Two Axes

Codex's safety model is **two independent flags**, not a tri-mode policy. Combine them.

### `-a, --ask-for-approval` (when to prompt the user)

| Value | Behaviour |
|-------|-----------|
| `untrusted` | Only "trusted" commands (e.g. `ls`, `cat`, `sed`) run without asking. Anything outside the trusted set escalates to the user. |
| `on-request` | The model decides when to ask the user. |
| `never` | Never asks; execution failures are returned to the model. |
| `on-failure` | **Deprecated** — prefer `on-request` for interactive runs or `never` for non-interactive runs. |

### `-s, --sandbox` (filesystem/network policy)

| Value | Behaviour |
|-------|-----------|
| `read-only` | Codex can read but not write. |
| `workspace-write` | Writes are confined to the workspace; broader filesystem and network restricted. |
| `danger-full-access` | No sandbox restrictions on filesystem/network. |

### The nuclear options

- `--dangerously-bypass-approvals-and-sandbox` — skip both. **Only** for environments that are externally sandboxed (CI in a fresh VM, throwaway containers). Pair with a `git worktree` if running locally.
- `--dangerously-bypass-hook-trust` — run enabled hooks without the persisted hook-trust prompt. Only for automation that already vets hook sources.

> [!warning] Pick from both axes
> A common mistake is setting only one of the two — e.g. `--ask-for-approval=never` without a tight `--sandbox` leaves you with no safety net. Reason about each axis separately.

> [!note] Check the machine's actual defaults before trusting the tier descriptions
> The values above are the **flag-level enum**, not what your shell will do by default. A user's `~/.codex/config.toml` can pre-set `approval = "never"` + `sandbox = "workspace-write"`, in which case a bare `codex exec "..."` runs without a single prompt. Every `codex exec` run prints its effective config in the header — verify before running anything destructive:
>
> ```bash
> codex doctor                         # full diagnostic
> codex exec "noop" 2>&1 | head -10    # see "model / approval / sandbox" header
> grep -E '^(approval|sandbox|model)'  ~/.codex/config.toml   # peek at config
> ```

## Config

User-level config: `~/.codex/config.toml` (this path is referenced in every `--help` block, so it's the canonical location). Project-level: typically `.codex/config.toml` at repo root.

Override any config key from the command line:

```bash
codex -c model="o3" exec "..."
codex -c 'sandbox_permissions=["disk-full-read-access"]' exec "..."
codex -c shell_environment_policy.inherit=all exec "..."
```

The value is parsed as TOML; if parsing fails, it's used as a literal string. Dotted paths (`foo.bar.baz`) override nested keys.

### Top-level CLI flags worth knowing

| Flag | Purpose |
|------|---------|
| `-m, --model <model>` | First-class model selection. |
| `-p, --profile <name>` | Pick a named profile from config.toml. **Note:** `-p` here is `--profile`, **not** `--print` like in Antigravity CLI. |
| `--profile-v2 <name>` | Layer a `$CODEX_HOME/<name>.config.toml` on top of base config. |
| `--oss` | Use the OSS-provider path. |
| `--local-provider lmstudio\|ollama` | Pick a local OSS provider. |
| `-C, --cd <dir>` | Use `<dir>` as the working root. |
| `--add-dir <dir>` | Additional writable directory alongside the primary workspace. |
| `-i, --image <file>` | Attach image(s) to the initial prompt. |
| `--search` | Enable live web search (native `web_search` tool, no per-call approval). |
| `--strict-config` | Error out on unknown config keys (catches typos in `~/.codex/config.toml`). |
| `--enable <feat>` / `--disable <feat>` | Toggle feature flags (`-c features.<name>=true/false`). |
| `--no-alt-screen` | Inline TUI; preserves scrollback. |
| `--remote <ws://…\|unix://…>` | Connect TUI to a remote app server. |

## AGENTS.md (shared convention)

Codex CLI reads `AGENTS.md` from the project root on every run and treats it as standing instructions. Same convention as Claude Code (via `CLAUDE.md` symlink), Antigravity CLI (`GEMINI.md`), and Kimi Coder (`KIMI.md`).

**For this workspace:** every agent under `agents/<name>/` already follows the pattern — `AGENTS.md` is the source of truth, with backend-specific filenames symlinked to it. Codex CLI picks up the agent's `AGENTS.md` automatically when run from inside an agent directory.

When editing `AGENTS.md` to support Codex:
- Keep it backend-neutral. Use `{{placeholder}}` for model IDs, tool names, paths.
- Codex respects the same headings/sections Claude does; no special markers needed.
- After editing, run the agent's `./scripts/check-agent-neutrality.sh` to verify no provider lock-in slipped in.

## MCP (Model Context Protocol)

Codex CLI is MCP-compliant. Management surface is **first-class subcommands**, not config-only:

```bash
codex mcp list           # list configured MCP servers
codex mcp get <name>     # inspect one
codex mcp add <name> …   # register a new server
codex mcp remove <name>  # remove
codex mcp login          # auth flow for OAuth-style MCP servers
codex mcp logout
```

You can also act **as** an MCP server: `codex mcp-server` runs Codex over stdio so other agents can call it as a tool.

Use the **same MCP server binaries** the user has installed for Claude Code / Antigravity — there's no Codex-specific MCP build needed.

## Plugins

Codex has a plugin marketplace model:

```bash
codex plugin list                                    # list plugins from configured marketplaces
codex plugin add <plugin-name>                       # install
codex plugin remove <plugin-name>
codex plugin marketplace add|list|upgrade|remove …   # manage configured marketplaces
```

Plugins are scoped to **OpenAI-curated marketplace snapshots** — no `import-from-claude` or `import-from-gemini` shortcut like Antigravity ships ([[../antigravity-cli/SKILL]]). If a user has Claude Code skills they want to port, plan a hand-rewrite.

## Codex Cloud (experimental)

`codex cloud` browses tasks from Codex Cloud and lets you apply changes locally. `codex apply <task-id>` is the local pull-down — it fetches the latest diff produced by a cloud agent run and applies it via `git apply` to your working tree.

This is the Codex equivalent of a "PR from your AI" workflow — review the diff after `codex apply`, edit if needed, then commit.

## Codex Desktop App

`codex app` launches the Codex desktop app (opens the app installer if missing). Codex has a desktop app — easy to miss if you only know the CLI.

## Common Workflows

### One-shot patch in an isolated worktree

```bash
git worktree add /tmp/refactor-auth -b refactor/auth
cd /tmp/refactor-auth
codex exec --sandbox workspace-write --ask-for-approval on-request \
  "extract the token-refresh logic from auth.ts into a separate module; keep the public API unchanged"
git diff
```

### Fully autonomous run in a sacrificial container

```bash
# In a fresh container / VM you're willing to throw away:
codex exec --dangerously-bypass-approvals-and-sandbox \
  "fix all of issue #4421; commit incrementally"
```

Never run that flag on a directory you can't afford to lose.

### Sandboxing an arbitrary command (without going through Codex)

```bash
codex sandbox macos -- pip install something-from-the-internet
codex sandbox linux -- ./some-build-script.sh
```

Useful as a generic seatbelt/bubblewrap wrapper — leverages Codex's existing sandbox profiles for non-Codex commands.

### Multi-LLM comparison

```bash
codex  exec "$(cat task.md)" > /tmp/codex-output.md
agy    -p   "$(cat task.md)" > /tmp/agy-output.md
claude -p   "$(cat task.md)" > /tmp/claude-output.md
diff3 /tmp/codex-output.md /tmp/agy-output.md /tmp/claude-output.md
```

Useful for the workspace's `agent-reviewer` to spot-check whether model choice is materially different on a given task.

### Running against a local OSS model

```bash
codex --oss --local-provider ollama   -m llama3.1:70b exec "explain this code"
codex --oss --local-provider lmstudio                 exec "fix the failing tests"
```

OSS-provider support is first-class via flags; you don't need to construct `OPENAI_BASE_URL` overrides like OpenClaude does ([[../../projects/openclaude]]).

### Resuming a previous session

```bash
codex resume --last                # most recent
codex resume                       # picker UI
codex resume <uuid>                # specific
codex resume <thread-name>         # named thread
```

`codex fork <id>` branches an existing session — useful for "what if I had answered differently?" without losing the original.

### Migrating from Claude Code

- `CLAUDE.md` → also read by Codex if symlinked to `AGENTS.md` (this workspace's convention already).
- `.claude/settings.json` permissions → not portable. Codex uses `--ask-for-approval` × `--sandbox` instead.
- `.claude/skills/<name>/SKILL.md` → no automatic import (unlike Antigravity, which ships `agy plugin import claude`). Either reference the skill content from `AGENTS.md` directly, or rewrite as a Codex plugin.
- MCP servers — fully portable; same server binaries work in both, and `codex mcp add` mirrors `claude mcp add`.

## Gotchas

- **`-p` is `--profile`, not `--print`.** Antigravity CLI uses `-p` for one-shot non-interactive runs; Codex uses `codex exec` for that and reserves `-p` for config profiles. Easy to muscle-memory the wrong tool.
- **`--dangerously-bypass-approvals-and-sandbox` + no isolation = lost work.** Always pair it with a throwaway worktree, container, or VM. The flag exists for externally-sandboxed environments, not for "I'll trust it just this once."
- **Approval mode is two flags, not one.** Setting `--ask-for-approval=never` without a tight `--sandbox` is a foot-gun. Reason about both axes.
- **Model selection has multiple entry points.** `-m/--model` flag, `-c model="..."`, or the `model` key in `~/.codex/config.toml`. Flag wins over config; check `codex --help` if a model isn't sticking.
- **`AGENTS.md` is read every run.** Long files inflate every prompt. Keep them tight; push lengthy context into linked files Codex can read on demand.
- **Codex does not auto-install MCP servers** — they must be installed and on PATH before being referenced via `codex mcp add`.
- **Project `.codex/config.toml` is NOT gitignored by default.** Decide explicitly whether to commit it (team-shared config) or gitignore it (per-developer).
- **`--strict-config` is your typo-catcher.** Useful when iterating on `config.toml`; otherwise unknown keys silently no-op.

## See Also

- [[../antigravity-cli/SKILL]] — Google's competing agentic CLI; same `AGENTS.md` convention, opposite `-p` semantics, has built-in `plugin import claude|gemini`.
- [[../../knowledge/google-antigravity]] — Antigravity product context; useful counterpoint when users are comparing.
- `agents/_template/AGENTS.md` — workspace's neutral agent protocol; already Codex-compatible.
- `agents/_template/neutrality.md` — why backend-neutral placeholders matter when multiple agentic CLIs read the same `AGENTS.md`.
