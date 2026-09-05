# Change: Add Answer Export to Markdown Files

## Why

Dexter's final answers currently exist only in terminal scrollback and an in-memory `MessageHistory` (`src/utils/message-history.ts:29`) that is discarded when the process exits. Users running financial research have no way to keep a result for later reference, sharing, or record-keeping.

## What Changes

- Add an export module (`src/utils/export.ts`) that serializes a completed turn — query, answer body, sources, and metadata — to a Markdown file.
- Automatically save every completed answer to `.dexter/reports/` using a timestamped, query-slugified filename.
- Surface the saved file path to the user via the existing `StatusMessage` component.
- Reuse the on-demand directory-creation pattern (`mkdirSync({ recursive: true })`) already used in `src/utils/context.ts:158` and `src/utils/config.ts`.
- Treat export failures as non-fatal: report a status message and keep the agent running.

## Impact

- Affected specs: `answer-export` (new capability)
- Affected code: `src/utils/export.ts` (new), `src/cli.tsx` (wire into `handleAnswerComplete`, ~line 159), tests under `src/utils/`
- `.gitignore` already ignores `.dexter/`, so `.dexter/reports/` is covered with no change
- Dependents: `add-export-output-modes` reuses this module for its `/save` command and one-shot mode
