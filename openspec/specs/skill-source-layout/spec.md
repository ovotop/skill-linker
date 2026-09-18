# skill-source-layout Specification

## Purpose
Defines where a skill's canonical source must live in a repository and how tool discovery
directories reach that source through relative symlinks, so that every CLI tool reading the
repo loads the same live copy from a single git-tracked location.

## Requirements

### Requirement: Canonical source lives in a tracked skills directory

Each skill's authoritative source SHALL be a directory under `skills/<skill-name>/`
containing its `SKILL.md`, and that directory SHALL be tracked by git rather than ignored.

#### Scenario: Source directory is tracked

- **WHEN** a skill's source has been placed under `skills/<skill-name>/`
- **THEN** `git ls-files skills/` lists that skill's files

#### Scenario: Source is not confined to a gitignored directory

- **WHEN** a skill would otherwise be stored only inside a gitignored directory such as `.opencode/skills/`
- **THEN** the skill is not considered correctly installed, because an ignored location provides no diff history and nothing to publish

### Requirement: SKILL.md declares name and description frontmatter

Each skill's `SKILL.md` SHALL begin with YAML frontmatter carrying at least a `name` and a
`description`, because tooling reads these two fields to identify the skill and to decide
when to load it.

#### Scenario: Skill file declares the required frontmatter

- **WHEN** a skill's `SKILL.md` is created under `skills/<skill-name>/`
- **THEN** the file opens with frontmatter containing both a `name` and a `description`

#### Scenario: Skill file omits the required frontmatter

- **WHEN** a `SKILL.md` lacks a `name` or a `description` in its frontmatter
- **THEN** the skill is not considered correctly authored, because tooling cannot identify it or decide when to load it

### Requirement: Tool discovery directories reach the source by relative symlink

A tool's discovery directory SHALL expose a skill through a symlink whose target is a
relative path pointing back to `skills/<skill-name>/`.

#### Scenario: Relative symlink resolves to the canonical source

- **WHEN** a skill is linked into a tool discovery directory
- **THEN** the link target resolves to `../../skills/<skill-name>` and the link is stored as a relative path, not an absolute one

#### Scenario: Repository remains portable after clone

- **WHEN** the repository is cloned onto another machine
- **THEN** the relative symlinks still resolve to the canonical source without any path rewriting

### Requirement: A single link serves all tools that read the shared discovery directory

Because `.claude/skills/` is read by both OpenCode and Claude Code, exposing a skill there
SHALL be sufficient for both tools; no separate `.opencode/skills/` entry is required.

#### Scenario: Both tools load the same live copy

- **WHEN** a skill is linked into `.claude/skills/`
- **THEN** both OpenCode and Claude Code resolve it to the same canonical source under `skills/`

### Requirement: Reserved generated directories stay ignored

`.opencode/skills/` and `.opencode/command/` SHALL remain gitignored, because tooling
regenerates them; durable, hand-maintained links SHALL live in `.claude/skills/` instead.

#### Scenario: Durable links are not placed in generated directories

- **WHEN** a durable skill link is being added
- **THEN** it is placed under `.claude/skills/` and not under `.opencode/skills/` or `.opencode/command/`

#### Scenario: Generated directories are not committed

- **WHEN** the ignore rules for this repository are inspected
- **THEN** `.opencode/skills/` and `.opencode/command/` are excluded from version control

### Requirement: Symlink creation degrades gracefully where symlinks are unavailable

On platforms where creating a symlink requires extra privilege or is otherwise unavailable,
the layout convention SHALL permit a documented fallback so the skill still becomes
discoverable.

#### Scenario: Platform cannot create symlinks

- **WHEN** symlink creation is not possible on the current platform
- **THEN** a fallback such as a junction link or a plain copy is used to expose the skill in the discovery directory
