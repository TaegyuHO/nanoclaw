## ADDED Requirements

### Requirement: AGENTS.md as universal entry point
The project root SHALL contain an `AGENTS.md` file that lists all available guides with their paths and one-line descriptions. This file serves as the discovery entry point for Codex, Copilot, and other AI tools that scan for `AGENTS.md`.

#### Scenario: Codex discovers guides
- **WHEN** Codex opens the project and reads `AGENTS.md`
- **THEN** it SHALL find a structured list of all guides with relative paths to each `guide.md`

#### Scenario: New guide added
- **WHEN** a new guide is added to `guides/`
- **THEN** `AGENTS.md` SHALL be updated to include the new guide entry

### Requirement: Guide index file
The system SHALL maintain a `guides/index.yaml` file containing machine-readable metadata for all guides: name, path, description, tags, and requires fields. This enables programmatic discovery and filtering.

#### Scenario: Filtering guides by tag
- **WHEN** a tool queries `guides/index.yaml` for guides tagged `channel`
- **THEN** it SHALL receive entries for all channel-related guides (e.g., add-telegram, add-slack, add-discord)

#### Scenario: Index consistency with guide directories
- **WHEN** a guide directory exists in `guides/`
- **THEN** a corresponding entry SHALL exist in `guides/index.yaml`

### Requirement: Tool-specific mapping files
The system SHALL support optional tool-specific mapping files that bridge guides to each tool's native format. Mappings SHALL NOT duplicate guide content — they SHALL reference guides by path.

#### Scenario: Copilot integration
- **WHEN** the project includes `.github/copilot-instructions.md`
- **THEN** it SHALL reference relevant guides from `guides/` rather than duplicating their content

#### Scenario: Claude Code integration
- **WHEN** Claude Code loads a SKILL.md
- **THEN** the SKILL.md SHALL reference its corresponding guide via a `guide` field in frontmatter or an inline `$ref` path
