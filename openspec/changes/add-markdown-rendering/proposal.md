# Change: Render Answers as Markdown in the Terminal

## Why

The final answer is forced to plain text by the system prompt (`src/agent/prompts.ts:236` — "Use plain text ONLY - NO markdown") and rendered through a single bare Ink `<Text>` node (`src/components/AnswerBox.tsx:47`). All structure therefore rides on improvised whitespace that terminal wrapping mangles. Financial answers — with comparisons, key figures, and source lists — read poorly as a result.

## What Changes

- Update the final-answer system prompt to **request** clean, compact Markdown (headings, bold key figures, tables for comparisons, lists, and a `## Sources` section) instead of forbidding markdown.
- Render the completed answer as formatted Markdown in `AnswerBox` using a terminal Markdown renderer.
- Preserve streaming UX: render plain text while the answer is streaming, then re-render the finalized Markdown on completion (terminal renderers cannot handle partial Markdown mid-stream).

## Impact

- Affected specs: `answer-rendering` (new capability)
- Affected code: `src/agent/prompts.ts` (~line 236 and `getFinalAnswerSystemPrompt` at line 253), `src/components/AnswerBox.tsx`, `package.json` (one new rendering dependency)
- Output format note: this changes the answer's content format to Markdown. Coordinate with `add-answer-export` so exported files store the Markdown form directly.
