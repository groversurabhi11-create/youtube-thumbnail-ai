# CLAUDE.md

## Project

Statistically Speaking — an AI-powered thumbnail generation pipeline for a
statistics-education YouTube channel. Full context (vision, brand, audience,
design philosophy, pipeline architecture, per-agent responsibilities) lives in
`project_overview.md` — **read it before doing any agent-prompt design work.**
It is the single source of truth; don't re-derive or restate it from memory,
re-read it, since it may be edited between sessions.

## Current Stage

Pre-code. **Prompt Version 1 for all 5 pipeline agents is complete** (as of
2026-07-14):

1. Teaching Architect — done (v1.1), `prompts/teaching_architect.md`
2. Metaphor Generator — done (v1), `prompts/metaphor_generator.md`
3. Illustrator — done (v1), `prompts/illustrator.md`
4. Image Generator — done (v1), `prompts/image_generator.md` — structurally
   different from the other four, see note below
5. Thumbnail Critic — done (v1), `prompts/thumbnail_critic.md`

No application code exists yet (no package.json, no build/test scripts,
nothing to run). Work in this phase is prompt design + manual testing —
simulate the agent in conversation against real topics, review output quality
against the prompt's own stated rules, and iterate wording — not software
implementation.

Image Generator is not a reasoning/JSON-in-JSON-out agent like the other
four — it's a faithful-execution contract layered over whatever
text-to-image model eventually gets wired in, since most image APIs don't
support a persistent system role the way chat models do. Read its own
MISSION and NOTE ON IMPLEMENTATION sections before assuming it behaves like
the others.

Next: more multi-topic testing of the full chain (so far only "p-value" has
run end-to-end through all 5 agents, as each prompt's own worked example;
"random experiment" has been run through Teaching Architect → Image
Generator but not yet Thumbnail Critic), or begin implementation.

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
- Metaphor Generator outputs exactly 3 ranked metaphors (matches the
  long-term vision's "Top 3 Visual Metaphors" step). Each metaphor must be
  distinct in both source domain AND underlying mechanism, not just a
  different object wrapping the same comparison. `universality` is a
  comprehension-only check ("will viewers recognize this?") — brand-tone /
  premium-fit judgment is explicitly reserved for the Thumbnail Critic.
- Illustrator processes all 3 metaphors into 3 parallel, rank-preserving
  illustration concepts — no selection step, since no agent in the pipeline
  is assigned a curation role. It also owns mood/lighting/palette direction
  as prose creative direction (not literal hex codes), since an
  "image-generation-ready description" requires it and no other agent
  claims it. Hard rule: no text anywhere in a scene, ever — visual_anchors
  that imply wording get reinterpreted as pure visual signals (e.g. a
  physical crack instead of a "FAIR COIN" label).
- Image Generator is a faithful-execution contract, not a creative-reasoning
  prompt. Fixed v1 technical parameters deliberately match real YouTube
  thumbnail requirements: 16:9, minimum 1280x720 (prefer 1920x1080), JPG/PNG/
  GIF/BMP only, under 2MB file size. One image per concept (3 per topic), no
  variation multiplier.
- Thumbnail Critic outputs strict JSON (not human prose) for consistency
  with the rest of the pipeline, even though a human may read it during the
  optional final Human Review stage. It hard-codes project_overview.md's
  "education beats sensationalism" guiding principle as a non-negotiable
  rule, not just a scoring criterion, and includes explicit
  scoring-calibration guidance to avoid LLM-judge score inflation.

## Testing approach so far

No automated tests exist. Validate a prompt by manually feeding it a handful
of real topics (e.g. Bayes' Theorem, Central Limit Theorem, Confidence
Interval, Expected Value, Simpson's Paradox), checking the output against the
prompt's own quality bar and boundaries, and iterating wording — before
considering that agent's v1 "done" and moving to the next one in the pipeline.

Testing done so far:
- Metaphor Generator: tested against Central Limit Theorem, Simpson's
  Paradox, and "random experiment" — found and fixed 3 real gaps (visual-
  anchor scene creep, diversity-vs-mechanism ambiguity, universality
  drifting into brand-tone judgment).
- Illustrator and Image Generator: tested against "random experiment,"
  chained from Metaphor Generator's output — held up with no fixes needed.
- Thumbnail Critic: not yet tested against a real topic beyond its own
  worked example.
- "p-value" has been run end-to-end through all 5 agents as each prompt's
  worked example, for pipeline traceability — that's fixed example content,
  not independent testing, and shouldn't be treated as validation on its own.
