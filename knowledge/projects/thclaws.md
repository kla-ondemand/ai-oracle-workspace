# thClaws

A native-Rust AI agent workspace. One binary exposes four surfaces — desktop GUI, CLI REPL, non-interactive single-turn, and a webapp — all sharing the same `Agent` loop, `Session`, and `ToolRegistry`.

**Repo:** [thClaws/thClaws](https://github.com/thClaws/thClaws) · **Local clone:** `projects/thClaws/`

## Differentiating Angle

Two things distinguish thClaws from every other "agent workspace" in this folder:

1. **Native Rust + GUI.** Most peers are Node/Python CLIs. thClaws is a Rust binary with a Tauri-style webview (frontend is React+Vite bundled into a single HTML file). Linux GUI needs `libwayland-client0` + `libwebkit2gtk-4.1-0` + `libsoup-3.0-0`; headless boxes use `--cli` or `-p`.
2. **Standards-first, not bespoke.** Uses MCP for tool servers, `AGENTS.md` for project instructions (the vendor-neutral standard adopted by Google/OpenAI/Factory/Sourcegraph/Cursor), `SKILL.md` with YAML frontmatter, `.mcp.json`. Your config is portable to/from any other agent that speaks the same standards.

## What's In The Box (the non-obvious parts)

- **KMS (Knowledge Management System):** per-project/per-user markdown wikis under `.thclaws/kms/<name>/pages/`. Agent gets a one-line table of contents every turn + `KmsRead/Search/Write/Append/Delete` tools. **No embeddings** — grep + read, following Karpathy's LLM-wiki pattern. `/dream` runs a side-channel agent that mines recent sessions, dedupes pages, surfaces new insights, writes a dated audit-trail page reviewable via `git diff`.
- **Three tiers of orchestration:**
  - `Task` tool — model-driven subagents that block the parent's turn. Own tool registry, recurses up to 3 levels.
  - `/agent <name> <prompt>` — user-driven concurrent side-channel. Fresh tokio task, parallel with main, never enters main's history, own cancel token. *"Use it when you know exactly what you want a specialist to do."*
  - **Agent Teams** — multiple thClaws *processes* coordinating through a shared mailbox + task queue, each in its own tmux pane + optional git worktree.
- **Plan mode** with per-step Approve/Cancel/Skip/Retry — same UX in GUI sidebar and REPL `/plan`.
- **Long-running loops:** `/loop` (fixed interval), `/goal` (audit-driven until done), `/goal --auto` (Ralph-style overnight builder).
- **Hooks:** 8 lifecycle events (`pre_tool_use`, `post_tool_use`, `permission_denied`, `session_start`, `pre_compact`, …) with timeout + SIGKILL guarantees.
- **Sessions as JSONL** under `.thclaws/sessions/` — git-friendly, grep-friendly, never opaque. `--resume last` or `--resume <id>`.
- **`!` shell escape** in the REPL — no tokens, no approval, no agent round-trip.
- **Document workflow:** native PDF/DOCX/PPTX/XLSX read+edit+create + image rendering tools, no separate file-conversion step.

## Provider Breadth

Anthropic native + Claude Agent SDK, OpenAI Chat Completions + Responses/Codex, Gemini & Gemma, DashScope (Qwen), DeepSeek, z.ai (GLM), NVIDIA NIM, NSTDA Thai LLM (OpenThaiGPT/Typhoon/Pathumma/THaLLE), OpenRouter, Agentic Press, Azure AI Foundry, Ollama (incl. Anthropic-compatible mode and Ollama Cloud), LMStudio, plus a generic `oai/*` slot for LiteLLM/Portkey/Helicone/vLLM/internal proxies. Auto-detected by model name prefix. `/model` swaps mid-session; `/provider` swaps whole provider.

**The NSTDA Thai LLM support is unusual** — none of the other projects in this workspace explicitly support it. Useful if any of this work needs Thai-language paths.

## Source Pointers

- `README.md` — full feature surface.
- `frontend/` — React + Vite GUI, built into a single HTML file.
- Rust source — `cargo build --release --features gui --bin thclaws`. Requires Rust 1.85+, Node 20+, pnpm 9+.
- `.thclaws/settings.json` (project) / `~/.config/thclaws/settings.json` (user) — every knob lives here. API keys default to OS keychain.

## Why It Matters For This Workspace

thClaws is the **standards-aligned native** option in this set. If the goal is "agent harness that doesn't lock me in", thClaws's `AGENTS.md`/`SKILL.md`/MCP commitment is the strongest. If the goal is "an agent for non-engineers" (PMs, researchers, legal, marketing), the Chat tab is explicitly built for that.

## See Also

- [[hermes-agent]] — same standards story (skills, MCP) but Python + self-improving loop.
- [[openclaude]] — same multi-provider story but a Claude Code fork, not a Rust workspace.
- [[openclaw]] — same "agent harness" framing but channel-first rather than GUI/REPL-first.
