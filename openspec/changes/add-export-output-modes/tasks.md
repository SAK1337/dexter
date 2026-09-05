## 1. Manual save command

- [ ] 1.1 Parse `/save [path]` in `handleSubmit` (`src/cli.tsx` ~line 204)
- [ ] 1.2 Export the most recent completed turn via the export module; honor an explicit path or fall back to the default reports location
- [ ] 1.3 Confirm the saved location via `StatusMessage`; if no answer exists yet, report "nothing to save" and take no action

## 2. One-shot output mode

- [ ] 2.1 Add argument parsing in `src/index.tsx` to detect a positional `<query>` plus `--output <file>`
- [ ] 2.2 Run the agent once without entering the interactive UI and write the answer via the export module
- [ ] 2.3 Exit with a success code on completion and a non-zero code on failure

## 3. Tests

- [ ] 3.1 Test `/save` path parsing and the no-prior-answer case
- [ ] 3.2 Test one-shot argument parsing and exit-code behavior
