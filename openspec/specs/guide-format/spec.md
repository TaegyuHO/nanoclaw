## ADDED Requirements

### Requirement: Guide directory structure
The system SHALL store guides in a top-level `guides/` directory. Each guide SHALL be a directory containing at minimum one `guide.md` file. The directory name SHALL use kebab-case matching the skill name (e.g., `guides/add-telegram/guide.md`).

#### Scenario: Single-file guide
- **WHEN** a guide has no supporting files
- **THEN** the guide directory SHALL contain only `guide.md`

#### Scenario: Guide with references
- **WHEN** a guide requires supporting documents (templates, examples, diagrams)
- **THEN** additional files SHALL be placed in the same guide directory alongside `guide.md`

### Requirement: Guide metadata via frontmatter
Each `guide.md` SHALL begin with YAML frontmatter containing: `name` (string, required), `description` (string, required), `tags` (string array, optional), `requires` (object, optional — declares runtime prerequisites like tools, runtimes, services).

#### Scenario: Minimal frontmatter
- **WHEN** a guide has no special requirements
- **THEN** frontmatter SHALL contain at minimum `name` and `description`

#### Scenario: Guide with prerequisites
- **WHEN** a guide requires specific tools (e.g., Docker, Node.js 20+)
- **THEN** frontmatter SHALL include a `requires` field listing each prerequisite with its version constraint

### Requirement: Tool-neutral markdown body
The guide body SHALL use pure markdown without tool-specific directives. It SHALL NOT reference Claude Code tools (Bash, Read, Write, AskUserQuestion) or any AI tool's execution primitives. Commands SHALL be written as fenced code blocks with language tags (e.g., ```bash).

#### Scenario: Shell commands in guide
- **WHEN** a guide includes CLI commands to execute
- **THEN** commands SHALL appear in fenced code blocks with appropriate language tags, not as tool invocations

#### Scenario: Decision points in guide
- **WHEN** a guide requires user input or decisions
- **THEN** the guide SHALL present options as markdown lists or tables, not as tool-specific prompt constructs

### Requirement: Structured sections
Each guide SHALL organize content into clearly headed sections. Recommended sections: Overview, Prerequisites, Steps, Troubleshooting. The guide SHALL be self-contained — a human or any AI tool can follow it without external context.

#### Scenario: Guide readability
- **WHEN** a developer reads a guide without any AI tool
- **THEN** the guide SHALL be fully understandable and actionable as standalone documentation
