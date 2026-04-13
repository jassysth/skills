---
name: skill-manager
description: |
  管理、分类和组织 Claude Code Skills 的技能。当用户说"整理 Skill"时触发。
  自动扫描 ~/.claude/skills 目录，找出新增的 Skill，按类别归档到对应的 GitHub 仓库分类目录，并全量同步到 GitHub。
  适用于需要批量管理多个 skills、保持本地和远程同步、组织 skill 分类结构的场景。
---

# Skill Manager

自动管理和同步 Claude Code Skills 的工具。

## 触发条件

当用户说 **"整理 Skill"** 或 **"整理我的 Skills"** 时执行。

## 工作流程

### 步骤 1: 扫描 Skills 目录

扫描 `~/.claude/skills` 目录下的所有 Skill：

```bash
ls -la ~/.claude/skills/
```

每个 Skill 以独立文件夹形式存在，包含 `SKILL.md` 文件。

### 步骤 2: 获取现有分类状态

检查 GitHub 仓库的分类结构：

```bash
gh repo clone jassysth/skills -- --depth=1 /tmp/skills-repo
ls -la /tmp/skills-repo/
```

### 步骤 3: 分类规则

根据 Skill 的描述和名称，自动归类到以下类别：

| 类别 | 关键词 | 目标目录 |
|------|--------|----------|
| **document** | document-skills, pdf, docx, pptx, xlsx, markdown, writing | `docs/` |
| **creative** | canvas, art, design, image, visual, poster | `creative/` |
| **development** | code, api, mcp, script, programming, debug | `dev/` |
| **productivity** | internal-comms, workflow, automation, productivity | `productivity/` |
| **ai** | claude-api, anthropic, agent, llm | `ai/` |
| **meta** | skill-creator, skill 相关的工具类 | `meta/` |
| **uncategorized** | 无匹配项 | `others/` |

分类逻辑：
- 读取每个 Skill 的 `SKILL.md`，提取 `description` 字段
- 根据关键词匹配确定类别
- 如果是新建 Skill（本地存在但 GitHub 上没有），标记为 **new**

### 步骤 4: 同步到 GitHub

1. **复制本地 Skills 到分类目录**（如果目标仓库是新建的）
2. **提交并推送**：

```bash
cd /tmp/skills-repo
git add .
git commit -m "Sync: $(date '+%Y-%m-%d %H:%M')"
git push
```

### 步骤 5: 输出报告

生成同步报告：

```
## Skill 整理报告

- 扫描时间: YYYY-MM-DD HH:MM
- 本地 Skills 总数: N
- 新增 Skills: [列表]
- 分类结果:
  - document: N
  - creative: N
  - development: N
  - productivity: N
  - ai: N
  - meta: N
  - uncategorized: N
- GitHub 同步状态: ✓ 完成
```

## 配置

默认配置（可按需修改）：

- **本地 Skills 目录**: `~/.claude/skills`
- **GitHub 仓库**: `jassysth/skills`
- **主分支**: `main`

## 注意事项

- 确保 `gh` 已认证：`gh auth status`
- Skill 文件必须包含 `SKILL.md` 才视为有效 Skill
- 分类关键词可根据实际需求扩展
