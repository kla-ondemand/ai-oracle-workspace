---
name: research
description: Externally-sourced research with persistence to research/. Web search + repo reading, source verification, structured writeup to a durable topic file.
when_to_use: User says "research", "look into", "investigate", "what's the state of", "compare X vs Y", or wants externally-sourced findings persisted. Distinct from /learning (session takeaways → knowledge/) and from manual notes (dated → notes/).
user-invocable: true
---

# Research Skill

Conduct externally-sourced research on a topic and persist the result as a durable, citation-backed entry under `research/`. The output is meant to be **refreshable** — when the topic is researched again later, the same file is updated in place with a bumped `researched_at` date.

## Storage rule

- **All research writes → `research/{topic-kebab}.md`** at the workspace root.
- Topic-organized, kebab-case ASCII filenames, **no date prefix in filenames**. The date lives in frontmatter (`researched_at`) and in each source's access line.
- Indexed by `research/README.md`. Append a one-line entry per topic.
- Never write research output to `notes/` (reserved for manual journal entries) or `knowledge/` (reserved for durable concepts derived from study, not from external sourcing).

## How `research/` differs from neighbors

| Dir | Content | Filename | Source of truth |
|-----|---------|----------|-----------------|
| `knowledge/` | Durable concepts, internalized | kebab, no date | Repo reading + reasoning |
| `notes/` | Dated journal entries | `YYYY-MM-DD-{title}.md` | Manual, point-in-time |
| `research/` | Externally-sourced findings, refreshable | kebab, no date | Web + external docs (always cited) |

If a finding is purely conceptual ("how OAuth2 works"), it belongs in `knowledge/`. If it's empirical and time-sensitive ("current pricing for Anthropic API as of date X", "comparison of agentic CLIs available today"), it belongs in `research/`.

## Modes

The user's phrasing decides the mode. If ambiguous, ask.

### New research (default for "research X", "look into Y", "what's the state of Z")

1. **Pick a filename** — `research/{topic-kebab}.md`. One topic per file.
2. **Check for an existing entry** — `ls research/` and grep for the topic. If it exists, switch to refresh mode (below) rather than creating a sibling.
3. **Gather sources** — `WebSearch` for the topic, then `WebFetch` the most authoritative results. Aim for 3–7 primary sources. Prefer official docs > vendor blogs > third-party writeups > forum posts.
4. **Cross-check** — if two sources contradict, note both in the writeup with citations, don't pick silently.
5. **Write the entry** using the template below.
6. **Update the index** — append `- [{topic}](./{topic-kebab}.md) — {one-line hook}` to `research/README.md` under the right section.

### Refresh (for "re-research…", "update findings on…", "is X still current?")

1. Read the existing `research/{topic-kebab}.md` and note its `researched_at` date.
2. Re-run searches; focus on what may have changed since that date.
3. Edit the file in place:
   - Bump `researched_at` to today.
   - Update outdated facts; keep an explicit **Changed since {prev date}** subsection so refresh diffs are visible.
   - Add new sources; don't remove old ones (they're historical context).
4. Do **not** create a new file or duplicate.

### Recall (for "what did I research about X", "show research on…")

1. Read `research/README.md`.
2. Grep `research/` filenames and content for the term.
3. Report: filename, `researched_at` date, one-line summary. Don't paste full contents unless asked.
4. Flag anything older than ~6 months as "may be stale — consider refresh".

## Write template

```markdown
---
title: {Topic}
researched_at: YYYY-MM-DD
aliases:
  - {alternate name 1}
tags:
  - research/{primary-topic}
  - {other tags}
---

# {Topic}

{One-sentence framing — what this research set out to answer.}

## Summary

{2–5 bullet points with the headline findings. This is what a reader scans first.}

## Findings

### {First conceptual section}

{Body. Cite inline as `([Source N](url))` referring to the Sources list below.}

### {Additional sections as needed}

Common sections: Landscape, Comparison, Pricing, Tradeoffs, Adoption signals,
Risks, Open questions.

## Open Questions

{Things this pass did not resolve. Pickup points for the next refresh.}

## Sources

1. [{Source title}]({url}) — accessed YYYY-MM-DD. {One-line on why this source.}
2. ...

## See Also

- [[research/{related-topic}]]
- [[../knowledge/{related-knowledge-entry}]] (if a concept entry exists)
```

## Rules

- **Cite everything externally-sourced.** Every empirical claim in `## Findings` must trace to a numbered entry in `## Sources`. If you can't cite it, drop it.
- **`researched_at` is mandatory** in frontmatter. It's the signal for staleness.
- Convert relative dates in source material ("last quarter", "recently") to absolute when writing — sources rot, but the date the source captured is fixed.
- Filenames are kebab-case ASCII. No spaces, no date prefix.
- One topic per file. If a session covers two distinct topics, write two entries and cross-link in `See Also`.
- Prefer **refreshing an existing entry** over creating a new one — the whole point of `research/` is that entries get re-visited.
- Don't write into `research/README.md` directly except as an index entry.
- If a topic is really conceptual (no external citation, just understanding the code), redirect to `/learning` → `knowledge/` instead.

## Workspace overrides

If a workspace's `CLAUDE.md` redefines where research output should land (e.g. "send all research to `notes/` instead"), follow the workspace. The default in this skill is `research/` at the workspace root.
