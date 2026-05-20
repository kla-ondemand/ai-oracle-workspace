# Research

Externally-sourced research artifacts — findings backed by citable sources, refreshable over time. Distinct from `knowledge/` (durable concepts derived from reading code) and `notes/` (dated manual journal entries).

Written by the `/research` skill (see [`skills/research/`](../skills/research/SKILL.md)). Format: one file per topic, kebab-case ASCII, no date prefix. The `researched_at` field in frontmatter records the last refresh date.

## Conventions

- **One topic per file.** `research/{topic-kebab}.md`.
- **Cite everything.** Every empirical claim in `## Findings` traces back to a numbered entry in `## Sources`.
- **`researched_at` is mandatory** in frontmatter — it's the staleness signal.
- **Refresh, don't duplicate.** When a topic is researched again, edit the existing file in place and bump `researched_at`.
- **Filenames** are kebab-case ASCII. No spaces. No date prefix (date lives in frontmatter).

## Index

_No entries yet._

<!--
Append entries here as research is added, grouped into sections. Suggested
section headings:

## Agentic CLIs
- [codex-cli-vs-claude-code](./codex-cli-vs-claude-code.md) — feature parity, sandbox & approval model, AGENTS.md convention.

## Models & Pricing
- [anthropic-api-pricing](./anthropic-api-pricing.md) — current rates, prompt caching economics, batch discount.

## Standards & Protocols
- [mcp-spec-landscape](./mcp-spec-landscape.md) — Model Context Protocol status, vendor adoption, gotchas.
-->
