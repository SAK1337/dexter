## Context

Answers are produced as a stream by `AnswerPhase.run()` and consumed by `AnswerBox`. The single point where the complete answer, the originating query, and the turn's tasks coexist is `handleAnswerComplete` in `src/cli.tsx:159`. No persistence of answers exists today; the codebase only writes settings, `.env`, and cached tool JSON.

## Goals / Non-Goals

- Goals: durable, human-readable record of every answer; zero user friction; non-fatal on failure.
- Non-Goals: manual `/save` command and non-interactive output mode (covered by `add-export-output-modes`); changing the answer's content format (covered by `add-markdown-rendering`).

## Decisions

- **Location:** `.dexter/reports/` — consistent with the existing `.dexter/` convention and already gitignored.
- **Trigger:** auto-save on completion. A research tool benefits from every run leaving an artifact; explicit-only saving risks losing results.
- **Filename:** `<ISO-timestamp>_<query-slug>.md` for chronological sorting and human recognizability.
- **Format:** Markdown, so files are readable as-is and improve automatically once `add-markdown-rendering` lands.
- **Failure handling:** catch and surface via `StatusMessage`; never throw into the agent loop.

## Risks / Trade-offs

- Auto-save can accumulate many files over time → Mitigation: timestamped names keep them sortable; cleanup can be a later enhancement if needed.
- Timestamp generation in tests must be deterministic → Mitigation: accept `timestamp` as a parameter to `exportAnswer` so tests inject a fixed value.

## Open Questions

- Should the answer's tasks be included in the exported file, or only the query + answer + sources? (Default: query + answer + sources + metadata; tasks omitted for readability.)
