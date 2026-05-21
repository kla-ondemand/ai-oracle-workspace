---
name: openclaude
description: Drive OpenClaude — a Claude Code-shaped coding CLI with the provider swapped out. Same prompts/tools/agents/MCP/slash-commands surface, pointed at OpenAI-compatible (OpenAI, OpenRouter, DeepSeek, Groq, Mistral, LM Studio, Hicap), Gemini, GitHub Models, Codex (OAuth or CLI auth), Ollama, Atomic Chat, or Bedrock/Vertex/Foundry. Includes per-agent model routing, DuckDuckGo / Firecrawl web search, and an optional headless gRPC server for embedding.
when_to_use: User mentions "openclaude", "open claude", "claude code with X provider", "claude code on Ollama / DeepSeek / Groq / OpenRouter", or asks to run a Claude-Code-style workflow against a non-Anthropic backend. Also triggers when the user wants to embed Claude Code-style agentic tooling into another app via gRPC, or route different agents to different models.
user-invocable: true
---

# OpenClaude Skill

OpenClaude is the Claude Code terminal workflow **with the provider swapped out**. Same surface (prompts, tools, agents, MCP, slash commands, streaming output), different plumbing. Use it when you want the Claude Code UX but the call goes to OpenAI, an Ollama instance on your laptop, DeepSeek, Groq, Mistral, LM Studio, GitHub Models, Codex (OAuth or CLI auth), Gemini, Atomic Chat, or Bedrock / Vertex / Foundry.

This skill is **verified against `projects/openclaude/README.md` and the entry script `bin/openclaude` as of 2026-05-21** — provider list, env-var contract, and `~/.openclaude.json` schema below are what the README documents today. Re-verify if you see install command divergence (the repo is mirrored to GitLawb and GitHub).

## When To Use

- The user wants Claude Code's UX against a non-Anthropic provider (cost, local-first, vendor preference, sovereignty).
- The user wants `/provider` profiles — guided setup per backend, saved so they can switch.
- The user wants different agents to call different models (`agentRouting` in `~/.openclaude.json`) — Plan on GPT-4o, Explore on DeepSeek, etc.
- The user wants to embed an agentic coding loop into their own app via the headless gRPC server.
- The user is comparing OpenClaude to Hermes Agent / Codex / Claude Code / Antigravity and needs the concrete shape.

Do **not** invoke this skill for:
- Authoring code inside `projects/openclaude/` — that's a study clone; treat as read-only per workspace policy unless the user explicitly says otherwise.
- General Claude Code usage (the original CLI) — use [[../claude-cli/SKILL]].
- Non-coding personal-assistant flows — use [[../hermes-agent/SKILL]] (Hermes), which has memory + messaging gateway. OpenClaude is a coding tool.

## Install & Start

```bash
# Install (npm-published artifact):
npm install -g @gitlawb/openclaude

# Verify ripgrep is on PATH first — OpenClaude depends on it:
rg --version || echo "install ripgrep, then retry"

# Start interactive session:
openclaude
```

If `dist/cli.mjs` is missing (source-build path), the `openclaude` shim prints how to build:

```bash
# Source build path:
cd projects/openclaude
bun install
bun run build
node dist/cli.mjs
# Or, without building:
bun run dev
```

## Provider Surface — The Whole Point

OpenClaude's central feature is provider flexibility. There are three routes to wire it up, in order of friendliness:

### 1. `/provider` slash command (recommended for humans)

Inside an `openclaude` session, type `/provider`. Guided wizard: pick provider, paste credentials, save as a named profile. Profiles persist per user.

For GitHub Models specifically: `/onboard-github` runs a tailored onboarding with saved credentials.

### 2. Universal env-var escape hatch (OpenAI-compatible)

For any `/v1`-compatible backend (OpenAI, OpenRouter, DeepSeek, Groq, Mistral, LM Studio, Hicap, etc.):

```bash
export CLAUDE_CODE_USE_OPENAI=1
export OPENAI_API_KEY=sk-...
export OPENAI_BASE_URL=https://api.openrouter.ai/v1   # or your endpoint
export OPENAI_MODEL=anthropic/claude-sonnet-4         # or any served model id
openclaude
```

Local Ollama is the same shape — point `OPENAI_BASE_URL` at `http://localhost:11434/v1`, drop the API key, set `OPENAI_MODEL=qwen2.5-coder:7b` (or whatever you've pulled).

Windows PowerShell uses `$env:NAME="value"` instead of `export NAME=value`.

### 3. `ollama launch openclaude --model X` (zero config)

```bash
ollama launch openclaude --model qwen2.5-coder:7b
```

Ollama sets `ANTHROPIC_BASE_URL`, model routing, and auth automatically. Works with any model Ollama has pulled — local or cloud.

### Full provider matrix

| Provider | Setup path | Notes |
|----------|-----------|-------|
| OpenAI-compatible | `/provider` or env vars | OpenAI, OpenRouter, DeepSeek, Groq, Mistral, LM Studio, any `/v1` server |
| Hicap | `/provider` or env vars | `api-key` auth; auto-discovers models from `/models`; Responses mode for `gpt-` models |
| Gemini | `/provider` or env vars | API key only |
| GitHub Models | `/onboard-github` | Interactive onboarding with saved credentials |
| Codex OAuth | `/provider` | Opens ChatGPT sign-in in browser; stores creds securely |
| Codex (CLI auth) | `/provider` | Reuses existing `~/.codex` credentials |
| Ollama | `/provider`, env vars, or `ollama launch` | Local, no API key |
| Atomic Chat | `/provider`, env vars, or `bun run dev:atomic-chat` | Local Model Provider; auto-detects loaded models |
| Bedrock / Vertex / Foundry | env vars | For supported cloud environments |

## Per-Agent Model Routing

Different agents can call different models. Configure in `~/.openclaude.json`:

```json
{
  "agentModels": {
    "deepseek-v4-flash": {
      "base_url": "https://api.deepseek.com/v1",
      "api_key": "sk-..."
    },
    "gpt-4o": {
      "base_url": "https://api.openai.com/v1",
      "api_key": "sk-..."
    }
  },
  "agentRouting": {
    "Explore": "deepseek-v4-flash",
    "Plan": "gpt-4o",
    "general-purpose": "gpt-4o",
    "frontend-dev": "deepseek-v4-flash",
    "default": "gpt-4o"
  }
}
```

When no routing match is found, the global provider remains the fallback.

> [!warning] `~/.openclaude.json` stores API keys in plaintext
> Do not commit it. Treat it like `.env` — keep it readable only by the user, exclude from any backup that's not encrypted.

## Web Search & Fetch

- **Default `WebSearch`** on non-Anthropic models uses **DuckDuckGo scraping** — free, but rate-limit-prone and subject to DuckDuckGo's ToS.
- **Anthropic-native** and Codex Responses backends keep the provider's native web search.
- **`WebFetch`** uses raw HTTP + HTML→Markdown by default — fails on JS-rendered sites.
- **Set `FIRECRAWL_API_KEY`** to switch to Firecrawl-powered search (paid, more reliable) and Firecrawl scrape for `WebFetch`. Free tier: 500 credits.

```bash
export FIRECRAWL_API_KEY=your-key-here
openclaude
```

## Headless gRPC — Embedding OpenClaude

OpenClaude can run as a gRPC service so other apps embed its agentic loop:

```bash
# In the openclaude project root:
npm run dev:grpc                # starts gRPC server on localhost:50051
npm run dev:grpc:cli            # test CLI client that talks to it
```

Config via env:

| Var | Default | Purpose |
|-----|---------|---------|
| `GRPC_PORT` | `50051` | gRPC listen port |
| `GRPC_HOST` | `localhost` | Bind address (use `0.0.0.0` to expose — only behind auth) |

The proto is at `src/proto/openclaude.proto` — generate clients in Python / Go / Rust / etc. directly from it. The server uses **bidirectional streaming** to push text chunks, tool calls, and `action_required` permission prompts.

This is OpenClaude's most-overlooked feature. If a user wants to wrap Claude-Code-style agency into their own UI or pipeline without forking the CLI, gRPC is the right surface — not stdin/stdout scraping.

## Provider Caveats

- Anthropic-specific features (prompt caching shape, certain tool extensions) may not exist on other providers. OpenClaude adapts where it can.
- Tool quality depends heavily on the model. Smaller local models can struggle with long multi-step tool flows.
- Some providers impose lower output caps; OpenClaude truncates gracefully but doesn't magic them away.
- Use models with strong tool/function calling support for best results — that means `gpt-4o`-class, Claude Sonnet, Gemini 2 Pro, DeepSeek v3 / v4, Qwen 2.5 Coder 14B+, and similar.

## Common Workflows

### Run a one-shot coding task on OpenRouter's cheapest tool-capable model

```bash
export CLAUDE_CODE_USE_OPENAI=1
export OPENAI_API_KEY=$OPENROUTER_API_KEY
export OPENAI_BASE_URL=https://openrouter.ai/api/v1
export OPENAI_MODEL=deepseek/deepseek-chat
openclaude
```

### Local-only Coding on a Laptop (No API Spend)

```bash
ollama pull qwen2.5-coder:7b
ollama launch openclaude --model qwen2.5-coder:7b
```

Air-gappable. Useful for sensitive codebases or while travelling.

### Split agents across models (cheap-explorer / smart-planner)

Configure `~/.openclaude.json` as in the routing example above. `Explore` agent runs on DeepSeek for breadth, `Plan` runs on GPT-4o for synthesis.

### Embed in your own app via gRPC

1. Start the server: `npm run dev:grpc`.
2. Generate a client from `src/proto/openclaude.proto` in your target language.
3. Open a bidirectional stream; send a prompt; consume text chunks, tool-call events, and `action_required` permission prompts; reply to the latter with allow/deny.

### Migrating from Claude Code

- Slash commands and core tool surface are the same — most muscle memory transfers.
- Provider setup is OpenClaude's own (`/provider`); Anthropic auth is one of many.
- `.claude/skills/<name>/SKILL.md` skills are not auto-imported — port manually or reference from project conventions.
- MCP servers: same binaries work; configure via OpenClaude's MCP surface.

## Gotchas

- **`CLAUDE_CODE_USE_OPENAI=1` flips the brain.** Without it set, even with `OPENAI_*` vars present, OpenClaude may stay on the Anthropic path. The variable is a switch, not a hint.
- **`OPENAI_MODEL` is required** in the env-var path — there is no implicit default. Forgetting it produces a confusing "no model selected" error.
- **`~/.openclaude.json` is plaintext.** Don't commit it. Don't symlink it into a shared dir.
- **DuckDuckGo `WebSearch` is fragile.** If your workflow depends on search, configure Firecrawl — DuckDuckGo blocks aggressive scraping.
- **Smaller local models choke on long tool loops.** If the agent stalls mid-task on a 7B model, retry with a 14B+ class or move to a cloud provider for that turn.
- **Ripgrep is a hard dependency.** Install before first run; `rg --version` must work.
- **`ollama launch` overrides `ANTHROPIC_BASE_URL`.** Useful, but means any other Claude-Code-style tool running in the same shell will also route to Ollama unless you scope the launch.
- **gRPC `GRPC_HOST=0.0.0.0` is dangerous without auth** — there's no built-in authentication on the gRPC stream. Keep it on `localhost` or front it with a reverse proxy that does authn.
- **Provider feature parity is approximate, not exact.** When a tool produces weird output, check whether the provider supports the underlying capability before blaming OpenClaude.
- **Mirrored on GitLawb and GitHub.** If you see two install paths, they're the same package; pick whichever your network reaches.

## See Also

- [[../../knowledge/projects/openclaude]] — workspace orientation summary (positioning vs OpenClaw / Hermes / thClaws).
- [[../claude-cli/SKILL]] — the original Claude Code CLI; OpenClaude inherits its UX.
- [[../codex-cli/SKILL]] — the OpenAI sibling; comparison point for the safety model and `AGENTS.md` convention.
- [[../hermes-agent/SKILL]] — Hermes Agent; different shape (memory-driven personal assistant with messaging gateway, not a coding-only CLI), explicit successor to OpenClaw.
- [[../../knowledge/projects/hermes-agent]] · [[../../knowledge/projects/openclaw]] · [[../../knowledge/projects/thclaws]] — adjacent projects in this workspace's study set.
- `projects/openclaude/src/proto/openclaude.proto` — authoritative gRPC schema for client generation.
