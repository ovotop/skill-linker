## Purpose

Defines the checks that confirm a skill's symlink points at the intended canonical source,
that the link actually resolves, and that the source is tracked by git, so a broken or
drifting installation is detected rather than assumed correct.

## ADDED Requirements

### Requirement: Link target is verified

Verification SHALL inspect the symlink and confirm its target is the expected relative path
back to the canonical source.

#### Scenario: Inspecting the link target

- **WHEN** the discovery directory is listed with link targets shown
- **THEN** the skill's link target is `../../skills/<skill-name>`

### Requirement: Link resolution is verified

Verification SHALL confirm the symlink resolves to a readable skill file rather than
dangling.

#### Scenario: Link resolves to a skill file

- **WHEN** the linked skill file path is tested for existence
- **THEN** the test succeeds, confirming `SKILL.md` is reachable through the link

#### Scenario: Link is dangling

- **WHEN** the symlink target does not exist
- **THEN** verification fails, indicating the source is missing or the link is stale

### Requirement: Source tracking is verified

Verification SHALL confirm the canonical source is tracked by git rather than ignored or
untracked.

#### Scenario: Source is tracked

- **WHEN** the tracked files under `skills/` are listed
- **THEN** the skill's source files appear in that listing

#### Scenario: Source is not tracked

- **WHEN** the skill's source files are absent from the git-tracked listing
- **THEN** verification fails, indicating the skill would drift without history or a means to publish
