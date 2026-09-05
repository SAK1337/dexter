## ADDED Requirements

### Requirement: Markdown Answer Generation

The system SHALL instruct the language model to produce the final answer as well-structured Markdown.

#### Scenario: Markdown requested in prompt

- **WHEN** the final-answer prompt is constructed
- **THEN** it directs the model to use Markdown structure including headings, emphasis, lists, tables, and a sources section

### Requirement: Terminal Markdown Rendering

The system SHALL render the completed answer as formatted Markdown in the terminal rather than raw markup.

#### Scenario: Formatted rendering on completion

- **WHEN** answer streaming completes
- **THEN** the answer is displayed with Markdown formatting (headings, emphasis, lists, tables) and no literal markup characters such as `**` or `#`

### Requirement: Streaming Readability

The system SHALL keep the answer readable while it is still streaming.

#### Scenario: Plain text during stream

- **WHEN** the answer is still streaming
- **THEN** the partial answer renders as plain text without broken or partial Markdown artifacts

#### Scenario: Re-render on finalize

- **WHEN** streaming finishes
- **THEN** the system re-renders the full answer with Markdown formatting applied
