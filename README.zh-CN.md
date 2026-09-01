# skill-linker

[English](README.md) · **中文**

一个用来总结「skill 的维护方法」的 skill:把 skill 的规范源码放进一个被 git 追踪的
`./skills/` 目录,再用相对符号链接把它暴露给各个 CLI 工具的发现目录,让 OpenCode 和
Claude Code 始终加载最新版本。

它取代了「把 skill 源码放在被 gitignore 的目录(如 `.opencode/skills/`)」这种脆弱
的做法。放在 gitignore 目录里的 skill 没有 diff 历史(难以调试)、没有可推送的内容
(无法发布),而且要保持最新只能靠手工复制,迟早会漂移过时。

本仓库附带一个**最小可用范例** —— [`skills/skill-demo/`](skills/skill-demo/SKILL.md) ——
它完全遵循下面的约定。请把它当作可直接复制的模板来参考,而不是研究 `skill-linker`
自身(其自引用的目录结构容易造成误解):

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

## 约定

```
<repo>/skills/<skill-name>/                          # canonical source — git-tracked
<repo>/.claude/skills/<skill-name> ──────► ../../skills/<skill-name>
```

- **单一真源** —— skill 源码在 `skills/<name>/`,提交进 git,因此有 diff 历史、可发布。
- **相对符号链接** —— 每个工具的发现目录都指回这个真源。
- **`.claude/skills/` 同时被 OpenCode 和 Claude Code 读取**,所以一个链接服务两者,
  不需要单独的 `.opencode/skills/` 条目。
- **其他 CLI 工具**(Cursor、Windsurf 等)按需在自己发现目录加一个指向
  `skills/<name>/` 的符号链接即可。
- **`.opencode/skills/` 与 `.opencode/command/` 保持 gitignore** —— 它们由 OpenSpec
  插件自动生成,不要在其中手工放置持久链接。`.claude/skills/` 才是持久的、被追踪的位置。

## 为什么用这个模式

- **可调试** —— 源码在 git 里,有 diff 历史和 `git blame`。
- **可发布** —— 推送仓库即发布 skill,不再被 gitignore。
- **始终最新** —— 符号链接永远指向活源码;改一次,所有工具立即看到,无需手工复制。
- **可移植** —— 相对目标(`../../`)在别人机器上 `git clone` 后依然有效。

## 安装(让 skill-linker 全局可用)

```bash
# OpenCode 全局
ln -s ~/Documents/skills/skill-linker/skills/skill-linker ~/.config/opencode/skills/skill-linker

# Claude Code 全局
ln -s ~/Documents/skills/skill-linker/skills/skill-linker ~/.claude/skills/skill-linker
```

## 步骤

1. 把 skill 源码移动(或创建)到被追踪的目录:

   ```bash
   mkdir -p skills
   mv <old-location>/<skill-name> skills/<skill-name>
   ```

2. 创建共享符号链接:

   ```bash
   mkdir -p .claude/skills
   ln -s ../../skills/<skill-name> .claude/skills/<skill-name>
   ```

3. 修正 `SKILL.md` 里硬编码的旧路径(它经常引用自己之前的位置):

   ```bash
   grep -rn "opencode/skills\|\.opencode" skills/<skill-name>/
   ```

4. 纳入 git 追踪:

   ```bash
   git add skills/ .claude/skills/
   git commit -m "relocate <skill-name> to tracked skills/ + symlink"
   ```

5. (可选)对于不读取 `.claude/skills/` 的 CLI 工具,在它自己的 skill 目录里加一个符号链接:

   ```bash
   ln -s <path-to>/skills/<skill-name> <tool-skill-dir>/<skill-name>
   ```

## 验证

- `ls -la .claude/skills/` —— 链接目标是 `../../skills/<skill-name>`。
- `test -f .claude/skills/<skill-name>/SKILL.md` —— 链接可解析。
- `git ls-files skills/` —— 源码已被追踪(不再是零文件)。

## 常见坑

- **过时的硬编码路径** —— `SKILL.md` 可能引用旧位置;把这些引用更新为新的
  `skills/<skill-name>/...` 路径。
- **绝对路径 vs 相对路径** —— 优先用相对符号链接,让仓库保持 clone 可移植。
- **Windows** —— 创建符号链接需要额外权限;可退回 `mklink /J` 或直接复制。
- **保留目录** —— `.opencode/skills/` 与 `.opencode/command/` 保持 gitignore;
  OpenSpec 插件会自动生成它们,不要在其中手工放置持久链接。
