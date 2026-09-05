## ADDED Requirements

### Requirement: Manual Save Command

The system SHALL provide a `/save` command that exports the most recent completed answer to a user-specified location.

#### Scenario: Save to an explicit path

- **WHEN** the user enters `/save ./my-report.md` after an answer has completed
- **THEN** the system writes the most recent answer to that path and confirms the location

#### Scenario: Save with no path provided

- **WHEN** the user enters `/save` with no argument
- **THEN** the system exports the most recent answer using the default reports directory and naming

#### Scenario: Save with no prior answer

- **WHEN** the user enters `/save` before any answer has completed
- **THEN** the system reports that there is no answer to save and takes no destructive action
