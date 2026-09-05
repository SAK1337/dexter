## ADDED Requirements

### Requirement: One-Shot Output Mode

The system SHALL support a non-interactive invocation that runs a single query and writes the answer to a file.

#### Scenario: Run a single query to a file

- **WHEN** Dexter is invoked as `dexter "<query>" --output report.md`
- **THEN** it processes the query once, writes the final answer to `report.md`, and exits without entering the interactive UI

#### Scenario: Exit code reflects success

- **WHEN** a one-shot run completes successfully
- **THEN** the process exits with a success status; on failure it exits with a non-zero status
