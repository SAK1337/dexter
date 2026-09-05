# Dexter — Codebase Assessment & Recommendations

**Date:** 2026-06-17
**Scope:** Two reported issues — (1) the final answer is poorly formatted in the console, and (2) the agent cannot export its output to a text/markdown file for later use.

This document is an assessment with prioritized recommendations. It does **not** change any code.

---

## 1. Executive Summary

Both concerns are confirmed and have clear, well-scoped root causes:

| Issue | Status | Root cause | Effort to fix |
|-------|--------|------------|---------------|
| Poor console formatting | **Confirmed** | Answer is forced to plain text by the prompt **and** rendered through a single bare Ink `<Text>` node with no markdown renderer | Small–Medium |
| No file export | **Confirmed — feature absent** | The final answer is only ever held in React state + in-memory `MessageHistory`; nothing writes it to disk | Small |

Neither issue is a deep architectural problem. The agent pipeline (Understand → Plan → Execute → Reflect → Answer) is sound. Both fixes live at the **presentation/output boundary** of the app, not in the agent core.

---

## 2. How Output Flows Today (Traced)

Understanding the data path is what makes the fixes obvious:

```
AnswerPhase.run()                 src/agent/phases/answer.ts
  └─ callLlmStream(...)           returns AsyncGenerator<string>
       │
       ▼
orchestrator.callbacks.onAnswerStream(stream)   src/agent/orchestrator.ts:223
       │
       ▼
useAgentExecution → setAnswerStream(stream)     src/hooks/useAgentExecution.ts:305
       │
       ▼
<AnswerBox stream={...} onComplete={...} />     src/cli.tsx:376
       │
       ├─ renders:  <Text>{content}{▌}</Text>   src/components/AnswerBox.tsx:47  ← FORMATTING happens here
       │
       └─ onComplete(answer) ──► handleAnswerComplete()  src/cli.tsx:159
                                   ├─ setHistory([... {query, tasks, answer}])   (React state, scrollback only)
                                   └─ messageHistory.addMessage(query, answer)   (IN-MEMORY array)   ← EXPORT would hook here
```

**Key fact:** `handleAnswerComplete` in `src/cli.tsx:159` is the single point where the *complete* answer, the originating *query*, and the *tasks* are all available together. This is the natural insertion point for an export feature.

---

## 3. Issue #1 — Console Formatting

### 3.1 Findings

1. **The prompt forbids markdown.** `src/agent/prompts.ts:236` in the final-answer system prompt:
   ```
   - Use plain text ONLY - NO markdown (no **, *, _, #, etc.)
   - Use line breaks and indentation for structure
   ```
   So all structure is carried by raw whitespace the model improvises.

2. **The renderer can't format anyway.** `src/components/AnswerBox.tsx:45-52` renders the entire answer as one node:
   ```tsx
   <Box flexDirection="column" marginTop={1}>
     <Text>{content}{isStreaming && '▌'}</Text>
   </Box>
   ```
   There is no markdown parser, no per-line styling, no headings, no tables, no border, and no horizontal padding. Compare `DebugSection` (`src/cli.tsx:70`), which *does* use `borderStyle="single"` and `paddingX` — the answer, the most important output, has the least visual treatment.

3. **Terminal wrapping mangles whitespace structure.** Ink reflows text to terminal width. Indentation- and line-break-based "structure" survives poorly under wrapping, especially for the "key numbers on separate lines" guidance the prompt requests.

### 3.2 Recommendations (choose a path)

**Path A — Plain-text polish (lowest effort, no new deps).**
Keep plain text, but give `AnswerBox` real visual structure:
- Wrap the answer in a bordered, padded `<Box>` (a distinct "answer card"), with a colored label/header (e.g. "Answer").
- Detect and style the trailing `Sources:` section separately (the prompt already standardizes it — `prompts.ts:241-251`), e.g. dimmed/colored URLs.
- Render line-by-line so numbered points and the sources list get consistent indentation independent of model whitespace.

**Path B — Real markdown rendering (recommended; better console *and* better exports).**
1. Flip the prompt at `prompts.ts:236` to *request* clean, compact markdown (headings, bold for key numbers, tables for comparisons, bulleted lists, a `## Sources` section).
2. Render markdown in the terminal with a renderer such as `marked` + `marked-terminal`, or an Ink-native markdown component. This yields bold, headings, lists, and **tables** — a natural fit for the financial comparisons this agent produces.
3. **Streaming caveat (important):** markdown renderers expect complete blocks; rendering half-written markdown mid-stream looks broken. Recommended approach: render **plain streaming text while `isStreaming` is true** (the current behavior), then **re-render the finalized markdown once `onComplete` fires**. `AnswerBox` already distinguishes these states via `isStreaming`, so the seam is already there.

This path is preferred because the same markdown that renders nicely in the console is exactly what you want to write to a `.md` file (Issue #2) — one content format serves both.

### 3.3 Trade-off to decide
Plain-text (Path A) is faster and dependency-free but caps how good the output can look. Markdown (Path B) is the better long-term answer and pays double (console + export), at the cost of one rendering dependency and handling the streaming/finalize split. **Recommendation: Path B**, because export is also requested and markdown unifies both.

---

## 4. Issue #2 — Export to Text / Markdown File

### 4.1 Findings

- **No export exists anywhere.** A search for file writes in `src/` returns only:
  - `src/utils/config.ts:38` — `.dexter/settings.json`
  - `src/utils/context.ts:158` — cached tool-result JSON in `.dexter/context/`
  - `src/utils/env.ts:110` — `.env` API keys

  None write the answer.
- **`MessageHistory` is in-memory only.** `src/utils/message-history.ts:29` — `private messages: Message[] = []`. History is lost on exit; it is *not* a persistence layer.
- **No command or flag exposes saving.** `handleSubmit` (`src/cli.tsx:204`) handles only `exit`, `quit`, and `/model`. There is no `/save`, no `--output`, no auto-save.

### 4.2 Recommendations

Add a small, self-contained export module and wire it in at the `handleAnswerComplete` seam.

**Proposed shape:**
1. **New util:** `src/utils/export.ts` exposing something like
   `exportAnswer({ query, answer, tasks, model, timestamp }) → filepath`.
   - Default destination: a `reports/` (or `.dexter/reports/`) directory, created on demand with `mkdirSync({ recursive: true })` — mirror the existing pattern in `context.ts`/`config.ts`.
   - Filename: timestamp + slugified query, e.g. `2026-06-17T14-32-05_aapl-vs-msft-revenue.md`.
   - File body (markdown): the query as an `#` heading, a metadata block (date, model), the answer, and the `Sources` section. If Path B is adopted, the answer is already markdown — write it verbatim.

2. **Triggering — pick one or combine:**
   - **Auto-save every answer** (zero friction; good default for a research tool — every run leaves an artifact). Print the saved path under the answer.
   - **`/save` command** in `handleSubmit` to export the most recent turn on demand (precise control). Easy to add alongside the existing `/model` handling.
   - **One-shot/non-interactive mode** (`dexter "query" --output report.md`) — most valuable for scripting/automation, larger change since the app is currently interactive-only.

3. **Surface the result.** After writing, show the absolute path via the existing `StatusMessage` component so the user knows where the file landed.

4. **Housekeeping.** Add the chosen reports directory to `.gitignore` (the repo already ignores `.dexter/`, so `.dexter/reports/` is covered automatically — a top-level `reports/` would need a new entry).

### 4.3 Trade-off to decide
Auto-save guarantees nothing is lost but can clutter the directory over many queries; `/save` keeps it clean but relies on the user remembering. **Recommendation: auto-save to `.dexter/reports/` by default, plus a `/save <path>` command** for choosing a custom location — low effort, covers both habits.

---

## 5. Suggested Sequencing

1. **Export module + auto-save** (`src/utils/export.ts`, wired at `cli.tsx:159`). Highest value, lowest risk, no UI rendering complexity. Delivers "save output for later" immediately.
2. **Markdown answer rendering** (prompt flip at `prompts.ts:236` + renderer in `AnswerBox.tsx`, with the stream-plain / finalize-markdown split). Improves the console and enriches the exported files written in step 1.
3. **`/save <path>` command + optional one-shot `--output` mode** for power users / automation.

Steps 1 and 2 are independent and can land in either order; doing export first means the formatting work in step 2 automatically upgrades the files produced in step 1.

---

## 6. Open Decisions for You

These shape the implementation and are genuinely your call:

1. **Console format:** plain-text polish (Path A) or full markdown rendering (Path B)?
2. **Export trigger:** auto-save, `/save` command, one-shot CLI flag — or a combination?
3. **Export location & format:** `.dexter/reports/` vs. a visible `reports/`; `.md` vs `.txt` (markdown recommended).
4. **Streaming during markdown:** accept the plain-while-streaming → markdown-on-complete approach, or keep plain text throughout and only export as markdown?

Once you choose, the changes are small and localized to: `src/utils/export.ts` (new), `src/cli.tsx` (wiring + optional command), `src/components/AnswerBox.tsx` (rendering), and `src/agent/prompts.ts` (format instruction).
