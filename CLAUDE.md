# CLAUDE.md

## Project

Statistically Speaking — an AI-powered thumbnail generation pipeline for a
statistics-education YouTube channel. Full context (vision, brand, audience,
design philosophy, pipeline architecture, per-agent responsibilities) lives in
`project_overview.md` — **read it before doing any agent-prompt design work.**
It is the single source of truth; don't re-derive or restate it from memory,
re-read it, since it may be edited between sessions.

## Current Stage

Pre-code. The active milestone is **Prompt Version 1 for each of the 5 pipeline
agents**, in priority order:

1. Teaching Architect — done (v1), `prompts/teaching_architect.md`
2. Metaphor Generator — not started
3. Illustrator — not started
4. Image Generator — not started
5. Thumbnail Critic — not started

No application code exists yet (no package.json, no build/test scripts,
nothing to run). Work in this phase is prompt design + manual testing —
simulate the agent in conversation against real topics, review output quality
against the prompt's own stated rules, and iterate wording — not software
implementation.

## Pipeline

Topic → Teaching Architect → Metaphor Generator → Illustrator → Image Generator
→ Thumbnail Critic → (optional) Human Review

Hard rule from `project_overview.md`: **no agent should perform another
agent's job.** Each agent's "Should NOT" list matters as much as its
"Responsibilities" list — enforce it strictly when drafting or reviewing
prompts. The most common failure mode to watch for is an agent quietly
drifting into a downstream agent's territory (e.g. Teaching Architect
inventing a visual/metaphor instead of staying purely conceptual).

## Prompt file conventions (established with Teaching Architect)

- One file per agent: `prompts/<agent_name>.md`.
- File structure: title + version + pipeline-position line, then the system
  prompt itself in a fenced code block, then a `## Changelog` section logging
  what changed between versions and why.
- Agent-to-agent output is **strict JSON only** — no prose, no markdown code
  fences — since the consumer is the next AI agent in the pipeline, not a
  human. (This may differ for agents whose output a human reviews directly,
  e.g. Thumbnail Critic — decide per agent.)
- Include one full worked example (input → output) inside the system prompt
  itself. This has been the highest-value addition for output consistency.
- State explicitly what the agent must NOT do, not just what it should do.

## Design decisions to remember

- Channel scope: statistics is the foundation/core identity (channel name
  "Statistically Speaking" stays as-is), but content will expand over time to
  mathematics, AI, and data science. Agent prompts should speak in terms of
  "topic" generically, not assume statistics-only, but keep the stats-rooted
  brand voice. `project_overview.md` and `prompts/teaching_architect.md` were
  both broadened for this (Teaching Architect is now v1.1).
- Teaching Architect's input is a plain topic string (e.g. `"p-value"`), not a
  video script or YouTube URL. Supporting a URL later requires a separate
  upstream step to resolve it into transcript text first — a system prompt
  alone can't fetch external content.
- Ambiguous/broad topics are auto-narrowed by the agent itself, not flagged
  for human review — the long-term vision is a fully automated pipeline with
  no human-in-the-loop mid-pipeline.
- `common_misconceptions` is structured as `{misconception, why_its_wrong}`
  pairs, not bare strings. Reuse this pattern in other agents where relevant
  (e.g. Thumbnail Critic's feedback structure).

## Testing approach so far

No automated tests exist. Validate a prompt by manually feeding it a handful
of real topics (e.g. Bayes' Theorem, Central Limit Theorem, Confidence
Interval, Expected Value, Simpson's Paradox), checking the output against the
prompt's own quality bar and boundaries, and iterating wording — before
considering that agent's v1 "done" and moving to the next one in the pipeline.
