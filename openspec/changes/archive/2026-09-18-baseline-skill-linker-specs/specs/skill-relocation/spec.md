## Purpose

Defines the ordered workflow for moving or creating a skill so that its canonical source
ends up in the tracked skills directory, is reachable from tool discovery directories, and
carries no stale references to its previous location.

## ADDED Requirements

### Requirement: Skill source is moved or created into the tracked directory

Relocating a skill SHALL place its source directory at `skills/<skill-name>/`, moving an
existing skill from its prior location or creating the directory when the skill is new.

#### Scenario: Existing skill is relocated

- **WHEN** a skill currently stored elsewhere is relocated
- **THEN** its source directory is moved to `skills/<skill-name>/`

#### Scenario: New skill is created in place

- **WHEN** a skill is being created for the first time
- **THEN** its source directory is created directly at `skills/<skill-name>/`

### Requirement: A symlink exposes the relocated skill to tooling

After the source is in place, a symlink SHALL be created in the tool discovery directory
pointing at the canonical source.

#### Scenario: Link created after source is placed

- **WHEN** the canonical source exists at `skills/<skill-name>/`
- **THEN** a relative symlink from `.claude/skills/<skill-name>` to that source is created

### Requirement: Stale hardcoded paths inside the skill are corrected

Relocation SHALL include checking the skill's own files for references to its previous
location and updating them to the new tracked path.

#### Scenario: Old location is referenced inside the skill

- **WHEN** the relocated skill's files still reference its prior directory
- **THEN** those references are updated to point at `skills/<skill-name>/`

#### Scenario: No stale references remain

- **WHEN** the relocation steps have completed
- **THEN** searching the skill's directory for its old location yields no remaining matches

### Requirement: The relocated skill is committed to version control

The relocation SHALL conclude by staging and committing both the skill source and its
symlink so the skill becomes part of the repository's history.

#### Scenario: Source and link are committed together

- **WHEN** the skill source and its symlink have been placed correctly
- **THEN** both `skills/` and the discovery-directory link are staged and committed

### Requirement: Additional tools can be linked on demand

The convention SHALL support linking the same canonical source into the discovery
directory of any CLI tool that does not read the shared discovery directory, without
duplicating the skill's source.

#### Scenario: A tool does not read the shared directory

- **WHEN** a target CLI tool discovers skills from a directory other than `.claude/skills/`
- **THEN** a symlink to `skills/<skill-name>/` can be added in that tool's discovery directory
