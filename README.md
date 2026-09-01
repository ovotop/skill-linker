# skill-linker

**English** · [中文](README.zh-CN.md)

A skill that codifies **how to maintain a skill**: keep the canonical source in a
git-tracked `./skills/` directory and expose it to each CLI tool's discovery directory
through relative symlinks, so OpenCode and Claude Code always load the latest version.

This replaces the fragile pattern of storing skill source inside a gitignored directory
such as `.opencode/skills/`. A skill in a gitignored directory has no diff history
(hard to debug), nothing to push (can't publish), and drifts out of date because keeping
it current means manual copying.

This repo ships a **minimal working example** — [`skills/skill-demo/`](skills/skill-demo/SKILL.md) —
that follows the exact convention below. Use it as a copy-paste template rather than
studying `skill-linker` itself (whose self-referencing layout can be confusing):

```
skill-linker/
├── skills/
│   ├── skill-demo/  ──────────────────────┐
│   │   └── SKILL.md                       │
│   └── skill-linker/  ───────────────────┐│
├── .claude/                              ││
│   └── skills/                           ││
│       ├── skill-demo ◄──────────────────┼┘
│       └── skill-linker ◄────────────────┘
└── README.md
```

## The convention

```
<repo>/skills/<skill-name>/                          # canonical source — git-tracked
<repo>/.claude/skills/<skill-name> ──────► ../../skills/<skill-name>
```

- **Single source of truth** — the skill source lives in `skills/<name>/`, is committed
  to git, and therefore has diff history and is publishable.
- **Relative symlinks** — each tool's discovery directory points back to it.
- **`.claude/skills/` is read by both OpenCode and Claude Code**, so a single symlink
  serves both — no separate `.opencode/skills/` entry is needed.
- **Other CLI tools** (Cursor, Windsurf, …) can add their own symlink pointing at
  `skills/<name>/` on demand.
- **Keep `.opencode/skills/` and `.opencode/command/` gitignored** — the OpenSpec plugin
  auto-generates them, so don't hand-place durable links there. `.claude/skills/` is the
  durable, tracked location.

## Why this pattern

- **Debug** — the source is in git, so you get diff history and `git blame`.
- **Publish** — pushing the repo publishes the skill; it is no longer gitignored.
- **Latest version** — a symlink always resolves to the live source; edit once and every
  tool sees the change immediately, with no manual copying.
- **Portable** — relative targets (`../../`) survive `git clone` on other machines.

## Install (make skill-linker globally available)

```bash
# OpenCode (global)
ln -s ~/Documents/skills/skill-linker/skills/skill-linker ~/.config/opencode/skills/skill-linker

# Claude Code (global)
ln -s ~/Documents/skills/skill-linker/skills/skill-linker ~/.claude/skills/skill-linker
```

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

- **Stale hardcoded paths** — `SKILL.md` may reference its old location; update those
  references to the new `skills/<skill-name>/...` path.
- **Absolute vs relative** — prefer relative symlinks so the repo stays clone-portable.
- **Windows** — creating symlinks needs extra privilege; fall back to `mklink /J` or a
  plain copy.
- **Reserved directories** — leave `.opencode/skills/` and `.opencode/command/`
  gitignored; the OpenSpec plugin auto-generates them, so do not hand-place durable
  links there.
