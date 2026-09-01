---
name: skill-linker
description: >-
  Maintain skill source files in a git-tracked `./skills/` directory and expose them
  through relative symlinks from `.claude/skills/`, so OpenCode and Claude Code always
  load the latest version. TRIGGER on any mention of: skill维护/存放/存储位置, skill链接,
  symlink a skill, link a skill, relocate/move a skill out of a gitignored directory,
  把 skill 放到 skills 目录, keep skills up-to-date, skill directory convention, or
  `.claude/skills` symlink setup. Use whenever creating a new skill, moving an existing
  skill, or fixing a skill that is untracked/drifting out of date.
---

# Skill Linker

Keep a skill's canonical source in a tracked `./skills/` directory and link it into
tool-specific discovery directories, so every CLI (OpenCode, Claude Code, and others)
reads the same live copy.

This replaces the fragile pattern of storing skill source inside a gitignored
directory such as `.opencode/skills/`. A skill in a gitignored directory has no diff
history (hard to debug), nothing to push (can't publish), and drifts out of date
because keeping it current means manual copying.

## The convention

```
<repo>/skills/<skill-name>/                     # canonical source — git-tracked
<repo>/.claude/skills/<skill-name> ──────► ../../skills/<skill-name>
```

- **One canonical source** lives in `./skills/<skill-name>/` and is committed to git.
- **Relative symlinks** in each tool's discovery directory point back to it.
- **`.claude/skills/` is read by both OpenCode and Claude Code**, so a single symlink
  serves both — no separate `.opencode/skills/` entry is needed.
- **Other CLI tools** (Cursor, Windsurf, …) can add their own symlink on demand.

## Why this pattern

- **Debug**: the source is in git, so you get diff history and `git blame`.
- **Publish**: pushing the repo publishes the skill — it is no longer gitignored.
- **Latest version**: a symlink always resolves to the live source; edit once and every
  tool sees the change immediately, with no manual copying.
- **Portable**: relative targets (`../../`) survive `git clone` on other machines.

## Steps

1. Move (or create) the skill source into the tracked directory:

   ```bash
   mkdir -p skills
   mv <old-location>/<skill-name> skills/<skill-name>
   ```

2. Create the shared symlink:

   ```bash
   mkdir -p .claude/skills
   ln -s ../../skills/<skill-name> .claude/skills/<skill-name>
   ```

3. Fix any hardcoded old paths inside `SKILL.md` (it often references its own previous
   location):

   ```bash
   grep -rn "opencode/skills\|\.opencode" skills/<skill-name>/
   ```

4. Track it:

   ```bash
   git add skills/ .claude/skills/
   git commit -m "relocate <skill-name> to tracked skills/ + symlink"
   ```

5. (Optional) For a CLI tool that does not read `.claude/skills/`, add a symlink into
   that tool's own skill directory:

   ```bash
   ln -s <path-to>/skills/<skill-name> <tool-skill-dir>/<skill-name>
   ```

## Verify

- `ls -la .claude/skills/` — the link target is `../../skills/<skill-name>`.
- `test -f .claude/skills/<skill-name>/SKILL.md` — the link resolves.
- `git ls-files skills/` — the source is tracked (no longer zero files).

## Pitfalls

- **Stale hardcoded paths**: `SKILL.md` may reference its old location — update those
  references to the new `skills/<skill-name>/...` path.
- **Absolute vs relative**: prefer relative symlinks so the repo stays clone-portable.
- **Windows**: creating symlinks needs extra privilege; fall back to `mklink /J` or a
  plain copy.
- **Reserved directories**: leave `.opencode/skills/` and `.opencode/command/`
  gitignored — the OpenSpec plugin auto-generates them, so do not hand-place durable
  links there. `.claude/skills/` is the durable, tracked location.
