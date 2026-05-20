---
title: Google Antigravity
aliases:
  - Antigravity
  - Antigravity 2.0
  - Antigravity CLI
tags:
  - topic/agentic-coding
  - topic/agent-platform
  - vendor/google
---

# Google Antigravity

Google's **agent-first development platform** — a four-surface stack (desktop app, CLI, SDK, Managed Agents in the Gemini API) that shares one agent harness and positions agent orchestration, not the editor, as the primary developer surface. Co-optimised with Gemini models (Gemini 3.5 Flash in particular). Launched as **Antigravity 2.0** at Google I/O 2026 on 2026-05-19.

## Why Antigravity Is Strategically Different

Most agentic-coding tools in 2026 (Claude Code, Cursor, Copilot, Codex) treat the **editor as home** and the agent as a guest inside it. Antigravity inverts this: the **agent orchestration surface is home**, and the editor/code/terminal are tools the agent uses. The bet shows up in three concrete ways:

- **Standalone desktop app**, not a VS Code extension or fork. The desktop app is the "central home for agent interaction" with parallel subagent execution baked into the layout.
- **Provider lock to Gemini**, in exchange for deep ecosystem reach across Google AI Studio, Android, and Firebase. Trade-off: vendor flexibility vs. integration depth.
- **Cheap+fast-model strategy.** Co-optimisation targets **Gemini 3.5 Flash** (not the flagship Pro tier), implying the architecture is built around fan-out across many cheap subagent calls rather than a few expensive flagship reasoning calls.

## The Four Surfaces

| Surface | Purpose | Notes |
|---------|---------|-------|
| **Antigravity Desktop App** | Central GUI; orchestrates multiple subagents in parallel; scheduled tasks for background automation; ecosystem hooks into AI Studio / Android / Firebase. | The product's primary surface. Voice command support is native. |
| **Antigravity CLI** | Lightweight, high-velocity command surface for creating/running agents without a GUI. | **Successor to Gemini CLI** (see [[#gemini-cli-sunset-2026-06-18]] below). Rewritten in **Go** for faster startup and async workflows. |
| **Antigravity SDK** | Programmatic access to the same agent harness Google uses internally; lets you define custom agent behaviours and host them on your own infrastructure. | "Optimised for Gemini models" — provider portability untested. |
| **Managed Agents (Gemini API)** | Single-API-call sandboxed Linux environment per agent run; the agent reasons, calls tools, and executes code in isolation. | Same product category as Anthropic Computer Use / OpenAI Operator-style tools, exposed as a normal API primitive. |

A fifth track — **Gemini Enterprise Agent Platform** — wraps the SDK + Managed Agents for enterprise deployment.

## Key Concepts

### Dynamic Subagents

The headline orchestration primitive. Subagents spawn dynamically during a task and execute in parallel within one orchestration environment. Closest analogues in this workspace's `knowledge/projects/`:

- [[projects/maw-js]] — tmux-windowed parallel agents with HMAC federation. Maw's parallelism is human-orchestrated; Antigravity's is agent-orchestrated.
- [[projects/openclaw]] — channel-based dispatch (~23 messaging channels). Different shape: channel-per-conversation, not subagent-per-task.

### Plugins (formerly Skills / Hooks / Subagents / Extensions)

Gemini CLI's four extension primitives — Agent Skills, Hooks, Subagents, Extensions — are consolidated under the new **"Antigravity plugin"** umbrella. **Compatibility is not 1:1 at launch.** Migration of an existing Gemini CLI extension may need manual conversion.

### Managed Execution Sandbox

Each Managed Agents API call provisions an isolated Linux environment with tool-use + code-execution capability. Open questions remain about:
- Network egress policy
- Filesystem persistence across calls
- Secret/credential handling
- Resource quotas per call

These details were not surfaced in the launch announcement and need verification before relying on them for production workloads.

## Gemini CLI Sunset (2026-06-18)

> [!warning] Consumer-tier deadline
> On **2026-06-18**, Gemini CLI and IDE extensions stop serving requests for **Google AI Pro, Ultra, and free tier** users. Antigravity CLI is the replacement; install before the deadline.

| Tier | What Happens |
|------|--------------|
| Free / Pro / Ultra (consumer) | Gemini CLI deprecated 2026-06-18. Must migrate to Antigravity CLI. |
| Standard / Enterprise license | **Gemini CLI continues indefinitely** with model updates. No forced migration. |

Migration mechanics:
- Skills / Hooks / Subagents / Extensions → Antigravity plugins (no 1:1 parity guarantee).
- Auth tokens, plugin folders, cached state — official migration tooling status unclear from launch materials.
- This workspace's root has `.antigravitycli/` (gitignored at `.gitignore:87`), indicating prior interaction with the v1 CLI — worth auditing its contents before the deadline.

## Pricing & Tier Structure

| Plan | Price | Antigravity Quota |
|------|-------|---|
| Google AI Pro | (pre-existing tier) | baseline |
| Google AI Ultra | **$100/mo** (new at I/O 2026) | **5× Pro quota** |
| Standard / Enterprise | (negotiated) | + retained Gemini CLI access |

The Ultra tier sits directly opposite Anthropic Max and OpenAI ChatGPT Pro. The 5× quota framing ("higher AI limits **in Antigravity**") signals where Google wants high-spend users to concentrate their consumption.

## Gotchas

- **"Optimised for Gemini" usually means hard-coded provider in practice.** Don't assume the SDK is model-agnostic without reading the docs.
- **Migration is not lossless.** Google's own announcement: *"there won't be 1:1 feature parity right out of the gate."* Plan for rework on any non-trivial Gemini CLI extension.
- **Provider lock-in is the cost of the integration depth.** Antigravity 2.0 is by far the deepest-integrated dev suite into Google's stack (AI Studio + Android + Firebase) — but you cannot swap the model layer the way you can with [[projects/openclaude]].
- **`Gemini 3.5 Flash` co-optimisation ≠ flagship-only.** Architectures built on Antigravity should expect subagent fan-out as the default cost shape, not flagship-call counts.

## Relation to This Workspace

- The workspace's own `agents/agent-*/` persona pattern (`agent-coder`, `agent-reviewer`, `agent-qa`/BoBo, `agent-data-science`) is conceptually adjacent to Antigravity's "dynamic subagents" but **persona-defined and provider-neutral**, not runtime-spawned and Gemini-bound.
- `.antigravitycli/` is gitignored at workspace root — prior v1 state lives on this machine; pre-deadline audit advised.
- If any future agent in `agents/` is configured with `primaryModel: gemini-*`, the AI Ultra tier becomes the realistic floor for multi-agent workflows.

## References

### Primary (Google)
- [Google I/O 2026 developer highlights (blog.google)](https://blog.google/innovation-and-ai/technology/developers-tools/google-io-2026-developer-highlights/)
- [Transitioning Gemini CLI to Antigravity CLI (developers.googleblog.com)](https://developers.googleblog.com/an-important-update-transitioning-gemini-cli-to-antigravity-cli/)
- [antigravity.google](https://antigravity.google/) — landing page (minimal content; coverage articles below are richer).

### Coverage
- [Google Launches Antigravity 2.0 at I/O 2026 (MarkTechPost)](https://www.marktechpost.com/2026/05/19/google-launches-antigravity-2-0-at-i-o-2026-a-standalone-agent-first-platform-with-cli-sdk-managed-execution-and-enterprise-support/)
- [Google launches Antigravity 2.0 with an updated desktop app and CLI tool at IO 2026 (TechCrunch)](https://techcrunch.com/2026/05/19/google-launches-antigravity-2-0-with-an-updated-desktop-app-and-cli-tool-at-io-2026/)
- [Google Antigravity 2.0 becoming full agentic development suite (9to5Google)](https://9to5google.com/2026/05/19/google-antigravity-agentic-developer-suite/)
- [Antigravity 2.0 Gets Real in Competing with Claude Code and Codex (TelecomTalk)](https://telecomtalk.info/antigravity-2-gets-real-in-competing-with/1007680/)

## See Also

- [[../notes/2026-05-20-google-antigravity-2]] — first-pass dated note from launch day; captures personal takeaways and open questions.
- [[projects/openclaude]] — opposite strategy: same UX, swap-the-provider. Antigravity is same-provider, swap-the-surface.
- [[projects/openclaw]] — channel-dispatched personal assistant; closest behavioural cousin to multi-subagent fan-out.
- [[projects/maw-js]] — tmux-windowed federated agents; pre-Antigravity prior art for parallel agent execution.
- [[projects/oracle-framework]] — ψ/ shared-soul philosophy; useful counterweight to Google's "one harness across all surfaces" framing.
- [[projects/hermes-agent]] — self-improving agent with closed learning loop; Antigravity has no announced built-in learning loop.
- [[projects/thclaws]] — native-Rust agent workspace, standards-first (MCP, AGENTS.md, SKILL.md); contrast to Antigravity's Google-specific stack.
