# Change: Add Manual Save Command and One-Shot Output Mode

## Why

Auto-save (from `add-answer-export`) covers passive capture, but power users need explicit control: saving the most recent answer to a chosen path, and running Dexter non-interactively to write a report file for scripting and automation. The app is interactive-only today — `handleSubmit` (`src/cli.tsx:204`) handles only `exit`, `quit`, and `/model`, and there is no command-line query/output mode.

## What Changes

- Add a `/save [path]` slash command that exports the most recent completed answer to a user-specified path, falling back to the default reports location when no path is given.
- Add a one-shot, non-interactive invocation mode: `dexter "<query>" --output <file>` runs a single query, writes the answer to the given file, and exits with a status code reflecting success.
- Both reuse the export module introduced by `add-answer-export`.

## Impact

- Affected specs: `answer-export` (manual save command), `cli-invocation` (one-shot mode)
- Affected code: `src/cli.tsx` (`handleSubmit` ~line 204), `src/index.tsx` (argument parsing and mode selection)
- Depends on: `add-answer-export` (export module). Implement after that change.
