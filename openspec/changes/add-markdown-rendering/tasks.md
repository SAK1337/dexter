## 1. Prompt

- [ ] 1.1 Update `FINAL_ANSWER_SYSTEM_PROMPT` in `src/agent/prompts.ts` (~line 236) to request compact Markdown with headings, emphasis, lists, tables, and a `## Sources` section
- [ ] 1.2 Keep the sources format consistent with what `add-answer-export` writes to file

## 2. Rendering

- [ ] 2.1 Add a terminal Markdown rendering dependency (e.g. `marked` + `marked-terminal`, or an Ink-native Markdown component)
- [ ] 2.2 Update `src/components/AnswerBox.tsx` to render plain text while `isStreaming` is true, then formatted Markdown once streaming completes
- [ ] 2.3 Preserve the streaming cursor (`▌`) only during streaming

## 3. Verification

- [ ] 3.1 Snapshot/unit-test that a completed answer renders without raw markup characters (no literal `**`, `#`)
- [ ] 3.2 Manually verify a comparison query renders a Markdown table correctly in the terminal
