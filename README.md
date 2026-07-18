# youtube-thumbnail-ai
This repository has all the prompts for the youtube thumbnail generation task for my channel statistically speaking.

## Pipeline
Topic → Teaching Architect → Metaphor Generator → Illustrator → Image Generator → Thumbnail Critic → (optional) Human Review

See `project_overview.md` for full project context and `CLAUDE.md` for current status.

## Prompts
Each agent's system prompt lives in `prompts/<agent_name>.md`, with a
`prompts/<agent_name>_v1.md` counterpart for each. As of 2026-07-18 the
`_v1` files for Illustrator, Image Generator, and Thumbnail Critic hold a
drafted enhancement (thumbnail title + channel wordmark text baked into the
generated image) not yet merged into the live files — check each file's
internal `Version:` line to see which is newer.
