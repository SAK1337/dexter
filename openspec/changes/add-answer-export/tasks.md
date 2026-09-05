## 1. Export module

- [ ] 1.1 Create `src/utils/export.ts` with an `exportAnswer({ query, answer, sources, model, timestamp, path? })` function that returns the written file path
- [ ] 1.2 Implement a filename helper combining an ISO-style timestamp with a slug derived from the query (`.md` extension)
- [ ] 1.3 Build the Markdown body: query as heading, metadata block (date, model), answer body, and a sources section
- [ ] 1.4 Create the reports directory on demand with `mkdirSync({ recursive: true })`, mirroring `src/utils/context.ts:158`

## 2. Wiring

- [ ] 2.1 Call `exportAnswer(...)` from `handleAnswerComplete` in `src/cli.tsx` (~line 159), where query, tasks, and answer are all in scope
- [ ] 2.2 Capture the active model and a timestamp at save time
- [ ] 2.3 Show the saved path via `StatusMessage`; on write failure show a non-fatal error message and continue

## 3. Tests

- [ ] 3.1 Unit-test filename slugification and timestamp formatting
- [ ] 3.2 Unit-test Markdown body construction, with and without sources
- [ ] 3.3 Test that a write failure is caught and does not throw into the agent loop
