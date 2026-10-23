# 技能市集

Claude Code 插件市集。

## 新增此市集

```
/plugin marketplace add usira-okay/Skills
```

## 安裝插件

```
/plugin install dev-workflow@usira
```

## 可用插件

| 插件 | 說明 |
| --- | --- |
| [dev-workflow](plugins/dev-workflow) | 提供日常開發工作流程技能，涵蓋 Git worktree 管理（[copse](plugins/dev-workflow/skills/copse/SKILL.md)）與提交訊息慣例（`branch-aware-commits`）。安裝指令：`/plugin install dev-workflow@usira`。 |

## 專案目錄

```
.claude-plugin/marketplace.json   # 市集中繼資料
plugins/
  dev-workflow/
    .claude-plugin/plugin.json    # 插件中繼資料
    skills/
      copse/
        SKILL.md                  # Git worktree 管理技能
      branch-aware-commits/
        SKILL.md                  # 提交訊息慣例技能
```

每個插件位於 `plugins/<name>/`，並包含自己的 `.claude-plugin/plugin.json`，
且須登錄於 `.claude-plugin/marketplace.json`。
