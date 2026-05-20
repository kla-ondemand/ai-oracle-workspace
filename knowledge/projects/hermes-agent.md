# Hermes Agent

Nous Research's self-improving AI agent. Python-based. Runs anywhere — laptop, $5 VPS, GPU cluster, or serverless (Modal/Daytona) that hibernates idle. Reaches you via terminal + messaging gateway (Telegram/Discord/Slack/WhatsApp/Signal).

**Repo:** [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) · **Local clone:** `projects/hermes-agent/`

## Differentiating Angle

The README is explicit: **"the only agent with a built-in learning loop."** Hermes doesn't just *use* skills — it creates them from experience, improves them during use, mines its own past conversations (FTS5 + LLM summarization), and builds a persistent dialectic model of the user (via [Honcho](https://github.com/plastic-labs/honcho)). The agent compounding on itself is the headline feature.

## What It Is (and Isn't)

- **Provider-agnostic:** Nous Portal, OpenRouter, NovitaAI, NVIDIA NIM, Xiaomi MiMo, z.ai/GLM, Kimi/Moonshot, MiniMax, Hugging Face, OpenAI, or any custom endpoint. Switch with `hermes model`.
- **Seven terminal backends:** local, Docker, SSH, Singularity, Modal, Daytona, Vercel Sandbox. Daytona + Modal are the serverless-persistence ones — your env hibernates when idle.
- **Built-in cron** (`/schedule add`) plus delivery to any messaging platform — daily reports, nightly backups, weekly audits all in natural language.
- **Subagent spawning** for parallel workstreams; can write Python that calls tools via RPC, collapsing pipelines into zero-context-cost turns.
- **agentskills.io compatible** — skills as open standard, not a bespoke format.

## Why It Matters For This Workspace

Hermes is the **explicit successor to [[openclaw]]**: it ships `hermes claw migrate` which detects `~/.openclaw` and imports SOUL.md, memories, skills, command allowlist, messaging settings, API keys, TTS assets, and AGENTS.md. If you ever wonder "where does the OpenClaw lineage go?", the answer is Hermes.

## Architectural Threads To Pull On

- **Closed learning loop:** read the Skills System + Memory + Insights docs together — they're what makes Hermes different.
- **Honcho integration** for dialectic user modeling is the durable user-profile mechanism. Look at how it's wired in.
- **Voice memo transcription** + **cross-platform conversation continuity** make the messaging gateway feel like a single mind across surfaces.
- **Termux extra**: the full `.[all]` install pulls Android-incompatible voice deps, so Termux gets a curated `.[termux]` extra — useful precedent for any "mobile-friendly subset" pattern.

## Source Pointers

- `README.md` — feature matrix, install paths, OpenClaw migration.
- `python/` — the agent.
- Docs at [hermes-agent.nousresearch.com/docs](https://hermes-agent.nousresearch.com/docs/) — full architecture, skills, memory, MCP.

## See Also

- [[openclaw]] — the predecessor; Hermes imports its state.
- [[openclaude]] — different shape: a Claude Code fork for coding, not a memory-driven personal agent.
- [[thclaws]] — also has skills/MCP/AGENTS.md standards; different implementation language (Rust) and surface (GUI + REPL).
