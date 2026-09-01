---
name: skill-demo
description: A minimal demo skill used to illustrate the skill-linker directory convention. It shows how a canonical skill source lives in `./skills/<name>/` and is exposed through a relative symlink from `.claude/skills/`. This skill does nothing real — it exists only as a working example you can copy when creating your own skill.
---

# Skill Demo

This is a **minimal example skill** that demonstrates the directory convention
documented by `skill-linker`:

```
skills/skill-demo/  ────────┐
.claude/skills/             │
    skill-demo ◄────────────┘
```

It intentionally does nothing useful. Copy its structure when you create a real skill:

1. Create `skills/<your-skill>/SKILL.md` with the `name` + `description` frontmatter.
2. Link it: `ln -s ../../skills/<your-skill> .claude/skills/<your-skill>`.
3. Replace this body with your own instructions.

## Hello, world

If this skill loads correctly, you can verify the symlink resolves with:

```bash
test -f .claude/skills/skill-demo/SKILL.md && echo "resolved"
```
