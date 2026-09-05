## Context

`AnswerBox` (`src/components/AnswerBox.tsx`) consumes an `AsyncGenerator<string>` and accumulates chunks into a single `<Text>` node. It already distinguishes a streaming phase from completion via the `isStreaming` flag and the `onComplete` callback — the natural seam for switching rendering modes. The prompt currently forbids Markdown, so both the producer and the renderer must change together.

## Goals / Non-Goals

- Goals: readable, structured answers in the terminal; tables for comparisons; clean source lists.
- Non-Goals: rich rendering of streaming partials; changing the agent pipeline; file export (separate change).

## Decisions

- **Renderer:** a terminal Markdown renderer (`marked` + `marked-terminal` is the proven, low-dependency choice; an Ink-native Markdown component is an alternative). Final selection during implementation.
- **Streaming strategy:** render raw plain text while `isStreaming`, then re-render the full Markdown once `onComplete` fires. Rationale: Markdown renderers expect complete blocks; rendering half-written Markdown shows broken artifacts.
- **Prompt flip:** change `prompts.ts:236` from "plain text ONLY" to a request for compact, structured Markdown including a `## Sources` section.

## Risks / Trade-offs

- A new dependency adds bundle/runtime weight → Mitigation: choose a single, well-maintained renderer; no framework.
- Re-rendering on completion causes a brief visual switch from plain to formatted → Acceptable; mirrors common streaming-CLI behavior.
- Model may emit Markdown the renderer handles imperfectly (nested tables) → Mitigation: prompt for compact structures; renderer degrades to readable text.

## Migration Plan

No data migration. The prompt and renderer change in the same change so plain-text and Markdown rendering never diverge.

## Open Questions

- Should code-fenced blocks be syntax-highlighted, or is plain monospace sufficient? (Default: plain monospace.)
