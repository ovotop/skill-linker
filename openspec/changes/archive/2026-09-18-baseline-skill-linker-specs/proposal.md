# Baseline: restore skill-linker specs from the existing skill

## Why

`openspec/specs/` is empty and has never contained anything — `openspec/` was initialized
on 2026-09-18 with only a bare `config.yaml`, and no spec was ever committed to git. The
behavior this repo already implements and documents (in `skills/skill-linker/SKILL.md` and
the two READMEs) therefore exists only as prose, with no machine-checkable contract.

This change reconstructs that prose into baseline specs so the existing convention is
captured as testable requirements rather than tribal knowledge. It records **current
truth** only; it does not introduce new behavior.

## What Changes

- Capture the canonical-source + symlink convention as a spec capability.
- Capture the skill relocation workflow as a spec capability.
- Capture the layout verification checks as a spec capability.
- No code, documentation, or layout changes — this change writes spec artifacts only.

Explicitly **out of scope** (deferred to a separate future change): the multi-tool
cross-compatibility model described in `/tmp/docs/agents-paths.md` (`.agents/skills/` as
primary path, whole-directory symlinks across Cursor/Kiro/Gemini/dsh, extended frontmatter
fields, and an `install.sh` fallback). The repository does not implement any of that today —
`.agents/`, `.cursor/`, `.kiro/`, `.gemini/`, and `.dsh/` are all absent — so those concerns
belong in a forward-looking delta, not in a baseline that must describe what exists.

## Capabilities

### New Capabilities

- `skill-source-layout`: Where a skill's canonical source lives, how tool discovery
  directories reach it via relative symlinks, which directories are reserved/gitignored,
  and the portability constraints on link targets.
- `skill-relocation`: The ordered workflow for moving or creating a skill into the tracked
  directory, linking it, correcting stale hardcoded paths, and committing it.
- `skill-layout-verification`: The checks that confirm a skill's link target is correct,
  the link resolves, and the source is tracked by git.

### Modified Capabilities

None — no existing capability has requirements to change (`openspec/specs/` is empty).

## Impact

- **Specs**: three new capability directories under `openspec/specs/` after archive.
- **Code / runtime**: none. This change adds no executable code.
- **Docs**: none changed. The READMEs and `SKILL.md` remain the human-facing source; the
  specs are derived from them and must stay consistent with them.
- **`.gitignore` fixed in this change**: the ignore rules for `.claude` and `.agents` were
  rewritten from whole-directory ignores to content ignores with an explicit negation, so
  `.claude/skills/` and `.agents/skills/` stay tracked while the rest of each directory
  remains ignored (including `commands/`). Without this, every *new* skill link added under
  `.claude/skills/` would have been silently untracked — the exact drift this convention
  exists to prevent. Previously-tracked links were unaffected either way.
- **Validation scope**: no `SKILL.md` was modified, so the skill-creator eval loop
  (test prompts → with-skill/baseline runs → grading → review viewer) had nothing to run
  against. Instead both skills were checked against that skill's structural rules — both
  pass (name matches directory, description 1–1024 chars, no Claude-Code-only frontmatter,
  body under 500 lines). Auditing the specs against the skill text surfaced one real gap,
  since closed: the requirement that `SKILL.md` carry `name` + `description` frontmatter.
