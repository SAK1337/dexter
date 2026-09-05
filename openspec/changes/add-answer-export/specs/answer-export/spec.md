## ADDED Requirements

### Requirement: Answer File Export

The system SHALL serialize a completed research turn to a Markdown file containing the originating query, generation metadata (timestamp and model), the answer body, and any sources used.

#### Scenario: Export a completed answer

- **WHEN** the agent finishes generating a final answer
- **THEN** the system writes a Markdown file containing the query, the answer body, the sources section, and metadata (date and model)

#### Scenario: Sources omitted when none used

- **WHEN** a completed answer references no external sources
- **THEN** the exported file omits the sources section without error

### Requirement: Automatic Answer Saving

The system SHALL automatically persist every completed answer to a reports directory without requiring user action.

#### Scenario: Auto-save on completion

- **WHEN** an answer finishes streaming
- **THEN** the system saves it to `.dexter/reports/` and reports the saved file path to the user

#### Scenario: Reports directory created on demand

- **WHEN** the reports directory does not yet exist at save time
- **THEN** the system creates it recursively before writing the file

### Requirement: Stable Export Filenames

The system SHALL generate collision-resistant, human-readable filenames for exported answers.

#### Scenario: Timestamped, slugified filename

- **WHEN** an answer for the query "AAPL vs MSFT revenue" is exported
- **THEN** the filename combines an ISO-style timestamp with a slug derived from the query and uses a `.md` extension

### Requirement: Non-Fatal Export Failure

The system SHALL continue operating when an export write fails.

#### Scenario: Failed write does not crash the agent

- **WHEN** writing the export file fails (for example, a permission error)
- **THEN** the agent surfaces a non-fatal status message and continues running
