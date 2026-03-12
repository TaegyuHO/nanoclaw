## ADDED Requirements

### Requirement: Knowledge extraction from SKILL.md
Each SKILL.md SHALL be split into two parts: (1) tool-neutral knowledge extracted to `guides/<name>/guide.md`, and (2) Claude Code execution logic retained in SKILL.md. The extraction SHALL preserve all domain knowledge — installation steps, architecture decisions, configuration details, troubleshooting tips.

#### Scenario: Extracting a channel skill
- **WHEN** `add-telegram` SKILL.md contains both Telegram setup knowledge and Claude Code tool invocations
- **THEN** Telegram setup knowledge (bot creation, token setup, Socket Mode config) SHALL move to `guides/add-telegram/guide.md`, and SKILL.md SHALL retain only Claude Code orchestration logic referencing the guide

#### Scenario: Skill with no extractable knowledge
- **WHEN** a SKILL.md contains only Claude Code execution logic (e.g., a simple wrapper around a CLI command)
- **THEN** no guide SHALL be created; the SKILL.md SHALL remain unchanged

### Requirement: Guide reference in SKILL.md
After extraction, SKILL.md SHALL reference its corresponding guide using a `guide` field in the YAML frontmatter pointing to the relative path (e.g., `guide: ../../guides/add-telegram/guide.md`). The skill body SHALL include a directive to read the guide for domain context.

#### Scenario: SKILL.md references guide
- **WHEN** Claude Code loads a refactored SKILL.md
- **THEN** it SHALL find the `guide` frontmatter field and read the referenced guide file for domain knowledge before executing skill steps

#### Scenario: Backward compatibility
- **WHEN** a SKILL.md with a `guide` reference is loaded by Claude Code
- **THEN** the skill SHALL function identically to the pre-extraction version — no change in behavior or output

### Requirement: Migration strategy
Skills SHALL be migrated incrementally — not all at once. High-value skills (channel integrations, setup, debug) SHALL be prioritized. Each migration SHALL be verified by confirming the SKILL.md still executes correctly after extraction.

#### Scenario: Incremental migration
- **WHEN** only 5 of 33 skills have been migrated
- **THEN** the 28 unmigrated skills SHALL continue to work exactly as before with no guide dependency

#### Scenario: Migration verification
- **WHEN** a skill is migrated (knowledge extracted to guide)
- **THEN** the skill SHALL be tested by invoking it in Claude Code and confirming identical behavior
