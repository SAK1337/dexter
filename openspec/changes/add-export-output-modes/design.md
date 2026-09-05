## Context

Dexter is currently interactive-only: `src/index.tsx` renders the Ink `CLI` component, and `handleSubmit` (`src/cli.tsx:204`) recognizes only `exit`, `quit`, and `/model`. Adding a one-shot mode changes the invocation model (interactive → also non-interactive), which is why this change carries a design note. Both features depend on the export module from `add-answer-export`.

## Goals / Non-Goals

- Goals: explicit save-to-path control; scriptable single-query runs that produce a file and a meaningful exit code.
- Non-Goals: auto-save (already covered); Markdown rendering (separate change); streaming output to stdout in one-shot mode.

## Decisions

- **`/save` semantics:** with a path, write there; without a path, reuse the default reports behavior. "Most recent answer" is the last completed turn already held in app state (`history` in `src/cli.tsx`).
- **One-shot detection:** presence of a positional query argument (and/or `--output`) at process start selects non-interactive mode in `src/index.tsx`, bypassing the Ink interactive render.
- **One-shot execution:** run the agent once, collect the full answer, write via the export module, then exit. Avoid mounting the interactive UI.
- **Exit codes:** `0` on success, non-zero on failure — required for automation.

## Risks / Trade-offs

- One-shot mode must fully await the answer stream before exiting → Mitigation: collect the stream to completion, then write and exit.
- Argument parsing could collide with existing flags → Mitigation: keep the surface minimal (`<query>` + `--output`); document precedence.

## Migration Plan

Additive. Interactive mode is unchanged when no positional query is supplied.

## Open Questions

- Should one-shot mode also echo the answer to stdout in addition to writing the file? (Default: write file only; optionally print the saved path.)
