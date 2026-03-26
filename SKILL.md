---
name: ops-skill-creator
description: Create, publish, and manage skills following our workspace conventions. Use when creating a new skill with our naming rules, publishing a skill to GitHub, updating the TOOLS.md skill registry, or auditing existing skills against our standards. Triggers on "按我们的规范建 skill"、"发布 skill"、"skill 命名检查"、"新建一个 ops/mem/agents/model skill". NOT for generic skill creation without our conventions — that's the built-in skill-creator.
---

# ops-skill-creator

Extends the built-in `skill-creator` with our workspace conventions. **Always read the built-in skill-creator SKILL.md first** (`/opt/homebrew/lib/node_modules/openclaw/skills/skill-creator/SKILL.md`) for base spec (frontmatter, directory layout, progressive disclosure, packaging). This skill adds our naming, publishing, and registry rules on top.

## Naming Convention

Prefix categories (kebab-case, directory name = frontmatter `name`):

| Prefix | Category | Examples |
|--------|----------|----------|
| `ops-` | Tooling & operations | ops-browser, ops-docx-gen, ops-llm-usage |
| `mem-` | Memory management | mem-runbook, mem-baseline |
| `agents-` | Agent orchestration | agents-coder, agents-research |
| `model-` | Model management | model-escalation, model-routing |

**Rules:**
1. Directory name **must** equal frontmatter `name` field
2. All lowercase, hyphens only, prefix required
3. Under 64 characters

## Script Placement Rule

**所有脚本必须放在所属 skill 的 `scripts/` 目录内，禁止散落在 `workspace/scripts/`。**

```
skills/{skill-name}/
├── SKILL.md
├── references/     # 参考文档、配置模板
└── scripts/        # 所有可执行脚本
    ├── bench.py
    └── setup.sh
```

**原则：**
1. **一个脚本只属于一个 skill** — 不重复存放，不在 workspace/scripts/ 留副本
2. **skill 目录是自包含的** — 拷贝整个目录到另一台机器就能用
3. **workspace/scripts/ 仅放临时性一次性脚本**（项目报告生成等），不放 skill 工具
4. **跨 skill 共享脚本** — 放在被引用最多的 skill 内，其他 skill 在 SKILL.md 里用相对路径引用

**审计命令：**
```bash
# 检查 workspace/scripts/ 里是否有应归入 skill 的脚本
for f in ~/.openclaw/workspace/scripts/*; do
  base=$(basename "$f")
  found=$(find ~/.openclaw/workspace/skills/*/scripts -name "$base" 2>/dev/null)
  [ -n "$found" ] && echo "⚠️  重复: $base → $found"
done
```

## Workflow: Create a New Skill

1. **Pick prefix** — determine which category (ops/mem/agents/model)
2. **Name** — `{prefix}-{verb-noun}` style, e.g. `ops-pdf-rotate`
3. **Scaffold** — create directory under `~/.openclaw/workspace/skills/{name}/`
4. **Write SKILL.md** — follow built-in skill-creator spec (frontmatter, body, progressive disclosure)
5. **Validate** — run the check in [Validation](#validation) below
6. **Update registry** — append to TOOLS.md `Skill 命名规范` table
7. **Commit** — `git add skills/{name} TOOLS.md && git commit`

## Workflow: Publish to GitHub

1. **Repo name**: `{skill-name}` (e.g. `ops-pdf-rotate`)
2. **Org**: `catgodtwno1`
3. **Create**:
   ```bash
   gh repo create catgodtwno1/openclaw-{skill-name} --public --source=skills/{skill-name} --push
   ```
4. **Update TOOLS.md** — fill in the GitHub column: `catgodtwno1/openclaw-{skill-name}`
5. **Commit registry update**

## Workflow: Audit Existing Skills

Run validation across all skills:

```bash
for f in ~/.openclaw/workspace/skills/*/SKILL.md; do
  dir=$(basename $(dirname "$f"))
  name=$(grep "^name:" "$f" | head -1 | sed 's/name: *//')
  # Check dir == name
  [ "$dir" != "$name" ] && echo "❌ MISMATCH: dir=$dir name=$name"
  # Check prefix
  echo "$dir" | grep -qE '^(ops|mem|agents|model)-' || echo "⚠️  NO PREFIX: $dir"
done
```

Check TOOLS.md registry is in sync with actual `skills/` directories.

## Validation

Before committing any new or renamed skill:

```bash
dir="{skill-name}"
name=$(grep "^name:" skills/$dir/SKILL.md | head -1 | sed 's/name: *//')
# 1. dir == name
[ "$dir" = "$name" ] && echo "✅ name match" || echo "❌ name mismatch"
# 2. has prefix
echo "$dir" | grep -qE '^(ops|mem|agents|model)-' && echo "✅ prefix ok" || echo "❌ missing prefix"
# 3. length
[ ${#dir} -lt 64 ] && echo "✅ length ok" || echo "❌ too long"
```

## Current Registry

See TOOLS.md → `Skill 命名规范` section for the authoritative list. Keep it updated after every create/rename/delete.
