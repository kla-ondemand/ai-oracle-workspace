---
title: Google Antigravity 2.0
aliases:
  - Antigravity 2.0
  - Antigravity CLI
tags:
  - topic/agentic-coding
  - vendor/google
  - source/io-2026
---

# Google Antigravity 2.0

**Date:** 2026-05-20
**Source:** [antigravity.google](https://antigravity.google/), Google I/O 2026 announcements, Google Developers Blog (Gemini CLI → Antigravity CLI transition note), MarkTechPost / TechCrunch / 9to5Google coverage.

## TL;DR

Google rebuilt **Antigravity** from an IDE-flavoured product (v1) into a **standalone agent-first development platform** (v2.0) launched at I/O 2026. It now ships as four surfaces over one shared agent harness: a **desktop app** (orchestrates multiple subagents in parallel), a Go-based **Antigravity CLI** (the explicit successor to Gemini CLI — consumer-tier Gemini CLI is sunset on **2026-06-18**), an **Antigravity SDK** (the same harness Google uses internally, self-hostable), and **Managed Agents in the Gemini API** (per-call isolated Linux sandboxes that reason + use tools + execute code). Co-optimised with **Gemini 3.5 Flash**. Monetisation moves up-market via a new **AI Ultra plan at $100/mo** with 5× higher Antigravity quota than Pro.

## What I Learned

- **Antigravity 2.0 is not an editor.** Google deliberately positioned it as "a central home for agent interaction" — agent orchestration is the primary surface, with editing/code as a downstream concern. This is a structurally different bet from VS Code+Copilot, Cursor, and Claude Code (which all treat the editor as the home and the agent as a guest).
- **Gemini CLI is being deprecated for consumers, not enterprises.** On 2026-06-18, Gemini CLI and IDE extensions stop serving Google AI Pro/Ultra and free-tier users. **Standard/Enterprise license holders keep Gemini CLI indefinitely** with continued model updates — Google is splitting consumer (forced migration) from enterprise (stable surface).
- **Antigravity CLI is rewritten in Go.** Faster startup, async workflows that don't lock the terminal during long agent runs, and shares the same agent harness as the desktop app — so improvements propagate across surfaces instead of bifurcating.
- **Existing Gemini CLI artefacts migrate as "Antigravity plugins."** Agent Skills, Hooks, Subagents, and Extensions are preserved as a concept but renamed — Google explicitly warns **"there won't be 1:1 feature parity right out of the gate."** So migration is mechanical-ish for common cases but not guaranteed lossless.
- **"Dynamic subagents" are the headline orchestration primitive.** Parallel agent execution within one task — closest analogue is OpenClaw's multi-channel dispatcher and maw-js's tmux-windowed federation, but here it's first-class and provider-locked to Gemini.
- **Managed Agents in the Gemini API = single-API-call sandboxed Linux env.** Each call spins an isolated environment where the agent reasons, calls tools, executes code. Same product category as Anthropic Computer Use and OpenAI's Operator-style offerings, but exposed as a normal API primitive rather than a separate product surface.
- **Co-optimisation target is Gemini 3.5 Flash**, not the flagship Pro model. Suggests Google is betting on cheap+fast subagent fan-out rather than fewer-but-smarter agent calls — the cost model favours quantity.
- **Ecosystem reach is broader than competitors.** Antigravity 2.0 hooks into Google AI Studio, Android (build/deploy), and Firebase — Claude Code and Codex are coding-tool-shaped; Antigravity is sliding toward "the dev suite for the whole Google stack."
- **Pricing signals what they think it's worth.** $100/mo Ultra = directly comparable to Anthropic's Max tier and OpenAI's ChatGPT Pro. Five-times quota multiplier vs Pro is generous; the framing of "in Antigravity" (not "in Gemini") signals where Google wants users to spend their tokens.
- **This workspace already has Antigravity CLI installed.** `.antigravitycli/` is gitignored at root (`.gitignore:87`) — so the local machine has interacted with the v1 CLI already. Worth checking what state survives the rename on 2026-06-18.

## Why It Matters

- **Agent-platform landscape shift.** `knowledge/projects/` documents six adjacent platforms (openclaude, openclaw, oracle-framework, hermes-agent, maw-js, thClaws) — Antigravity 2.0 is now another major entrant, with the deepest vendor integration (Google AI Studio + Android + Firebase) but the strongest provider lock-in (Gemini-only). Worth a `knowledge/agentic-platforms-2026.md` consolidation entry later, once enough of these have been studied at the same depth.
- **Concrete migration deadline on this machine.** The local `.antigravitycli/` was usable yesterday — after 2026-06-18 the consumer-tier Gemini CLI surface stops serving requests. If any local agents or scripts depend on `gemini` CLI commands, they need to switch to `antigravity` CLI before that date.
- **Parallel to the workspace's own structure.** The workspace's `agents/agent-*/` persona pattern (agent-coder, agent-reviewer, agent-qa/BoBo, agent-data-science) is conceptually close to Antigravity's "dynamic subagents." The migration semantics — Skills/Hooks/Subagents → Antigravity plugins — are worth a closer read before any local agent is ever ported across.
- **Pricing reframes the workspace's choices.** If the workspace's primary models stay Anthropic (Claude Opus 4.7 / Sonnet 4.6), Antigravity Ultra is irrelevant. If any future agent here is configured `primaryModel: gemini-3.5-flash`, the $100/mo plan is the realistic floor for serious multi-agent fan-out.

## Open Questions

- **What happens to `.antigravitycli/` state across the rename?** Plugins, auth tokens, cached agents — Google's note says "Antigravity plugins" but it's unclear whether existing Gemini CLI plugin folders are auto-migrated or need manual import.
- **Is the SDK actually open enough to self-host non-Gemini models?** "Optimised for Gemini models" usually means hard-coded provider in practice. Worth a closer read of the SDK docs before assuming model-agnostic.
- **How does "Managed Agents" sandbox isolation compare to Anthropic Computer Use?** Linux env, single-API-call lifecycle — but network policy, fs persistence, secret handling all unspecified in the launch blog.
- **Does the agent-qa / BoBo persona protocol map cleanly onto an Antigravity subagent?** Specifically: severity ladder + verdict structure + worktree-isolated review — these are persona conventions in this workspace. Antigravity subagents may impose their own report shape.

## See Also

- [[../knowledge/projects/openclaude]] — Claude Code fork that swaps providers under the same UX; the inverse strategy to Antigravity's "one provider, multiple surfaces."
- [[../knowledge/projects/openclaw]] — personal AI assistant routed across ~23 channels; closest in spirit to "multiple agents executing tasks in parallel" but channel-shaped rather than subagent-shaped.
- [[../knowledge/projects/maw-js]] — tmux-windowed multi-agent runner with HMAC-federated addressing; the workspace's existing reference for parallel agents.
- [[../knowledge/projects/oracle-framework]] — ψ/ shared-soul philosophy; useful counterweight to Google's "one harness across all surfaces" framing.
- [[../knowledge/projects/hermes-agent]] — Nous Research's self-improving agent; closest comparable on the "closed learning loop" axis (Antigravity is open on this — no built-in learning loop announced).
- [[2026-05-20-workspace-bootstrap]] — earlier note from today; mentions `.antigravitycli/` in gitignore.

## Sources

- [Google I/O 2026 developer highlights — Antigravity, Gemini API, AI Studio (blog.google)](https://blog.google/innovation-and-ai/technology/developers-tools/google-io-2026-developer-highlights/)
- [Transitioning Gemini CLI to Antigravity CLI (developers.googleblog.com)](https://developers.googleblog.com/an-important-update-transitioning-gemini-cli-to-antigravity-cli/)
- [Google Launches Antigravity 2.0 at I/O 2026: A Standalone Agent-First Platform (MarkTechPost)](https://www.marktechpost.com/2026/05/19/google-launches-antigravity-2-0-at-i-o-2026-a-standalone-agent-first-platform-with-cli-sdk-managed-execution-and-enterprise-support/)
- [Google launches Antigravity 2.0 with an updated desktop app and CLI tool at IO 2026 (TechCrunch)](https://techcrunch.com/2026/05/19/google-launches-antigravity-2-0-with-an-updated-desktop-app-and-cli-tool-at-io-2026/)
- [Google Antigravity 2.0 becoming full agentic development suite (9to5Google)](https://9to5google.com/2026/05/19/google-antigravity-agentic-developer-suite/)
- [Antigravity 2.0 Gets Real in Competing with Claude Code and Codex (TelecomTalk)](https://telecomtalk.info/antigravity-2-gets-real-in-competing-with/1007680/)
