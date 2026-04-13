---
name: skill-manager
description: |
  管理、分类和组织 Claude Code Skills 的技能。当用户说"整理 Skill"时触发。
  自动扫描 ~/.claude/skills 目录，找出新增的 Skill，按类别归档到对应的 GitHub 仓库分类目录，并全量同步到 allskill 仓库。
  最后依次对所有仓库执行 git commit + push。
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

跳过以下目录：
- `.git` — Git 仓库目录
- `obsidian-skills` — 外部插件包
- 任何不含 `SKILL.md` 的目录

### 步骤 2: 获取现有仓库状态

检查已存在的分类仓库：

```bash
gh api "/users/jassysth/repos?per_page=100" --paginate --jq '.[].name'
```

### 步骤 3: 分类规则

根据 Skill 的 `description` 字段，自动归类到以下类别：

| 类别 | 关键词 | 仓库名 |
|------|--------|--------|
| **writing** | document-skills, docx, pdf, pptx, xlsx, writing, markdown, 写作 | `writing-skill` |
| **drawing** | canvas, art, design, image, visual, poster, 画图 | `drawing-skill` |
| **info** | fetch, search, web, 信息, 获取 | `info-skill` |
| **PM** | internal-comms, workflow, productivity, pm, 项目 | `pm-skill` |
| **other** | 无匹配项 | `other-skill` |
| **allskill** | 全部 Skills（主仓库） | `allskill` |

分类逻辑：
1. 读取每个 Skill 的 `SKILL.md`，提取 `description` 字段
2. 根据关键词匹配确定类别
3. 标记 **新增** 的 Skills（本地存在但所有分类仓库中都没有）

### 步骤 4: 创建缺失的分类仓库

如果分类仓库不存在，创建它：

```bash
gh repo create {category}-skill --public --description "Claude Skills: {category}"
```

### 步骤 5: 同步 Skills 到各仓库

对于每个分类：

1. **克隆或更新目标仓库**：
```bash
gh repo clone jassysth/{category}-skill /tmp/{category}-skill -- --depth=1
```

2. **复制 Skills 到分类目录**：
```bash
cp -r ~/.claude/skills/{skill-name} /tmp/{category}-skill/
```

3. **提交并推送**：
```bash
cd /tmp/{category}-skill
git add .
git commit -m "Sync: $(date '+%Y-%m-%d %H:%M')"
git push
```

### 步骤 6: 全量同步到 allskill 主仓库

```bash
gh repo clone jassysth/allskill /tmp/allskill -- --depth=1
cp -r ~/.claude/skills/*/SKILL.md /tmp/allskill/ 2>/dev/null || true
cd /tmp/allskill
git add .
git commit -m "Full sync: $(date '+%Y-%m-%d %H:%M')"
git push
```

### 步骤 7: 依次执行 (按仓库顺序)

按以下顺序依次执行 git commit + push：
1. `writing-skill`
2. `drawing-skill`
3. `info-skill`
4. `pm-skill`
5. `other-skill`
6. `allskill` (主仓库)

## 配置

- **本地 Skills 目录**: `~/.claude/skills`
- **GitHub 用户名**: `jassysth`
- **分类仓库后缀**: `{category}-skill`
- **主仓库名**: `allskill`
- **主分支**: `main`

## 输出报告

```
## Skill 整理报告

- 扫描时间: YYYY-MM-DD HH:MM
- 本地 Skills 总数: N
- 新增 Skills: [列表]
- 分类结果:
  - writing: N (仓库: writing-skill)
  - drawing: N (仓库: drawing-skill)
  - info: N (仓库: info-skill)
  - PM: N (仓库: pm-skill)
  - other: N (仓库: other-skill)
- 各仓库同步状态:
  - writing-skill: ✓/✗
  - drawing-skill: ✓/✗
  - info-skill: ✓/✗
  - pm-skill: ✓/✗
  - other-skill: ✓/✗
  - allskill: ✓/✗
```

## 注意事项

- 确保 `gh` 已认证：`gh auth status`
- Skill 文件必须包含 `SKILL.md` 才视为有效 Skill
- 分类关键词可根据实际需求扩展
- 按顺序执行推送，确保每个仓库成功后再处理下一个
