---
description: "对 GitHub Pull Request 进行自动化代码审查"
argument-hint: "[REPO: owner/repo PR_NUMBER: number]"
---

# /review-pr

对 GitHub Pull Request 进行全面的自动化代码审查，包括代码质量和架构设计分析。

## 输入参数

- `REPO`: 仓库名称，格式为 `owner/repo`
- `PR_NUMBER`: Pull Request 编号

## 审查流程

### 步骤 1: 获取 PR 信息

首先获取 PR 的详细信息和代码变更：

```bash
# 获取 PR 详情
gh pr view $PR_NUMBER --repo $REPO --json number,title,body,author,baseRefName,headRefName,files,additions,deletions,changedFiles

# 获取代码变更
gh pr diff $PR_NUMBER --repo $REPO
```

### 步骤 2: 并行分析

使用 subagent 并行分析以下维度：

#### 分析 A - 代码质量

分析代码变更中的：

- 代码风格一致性
- 代码复杂度
- 重复代码
- 命名规范
- 错误处理
- 潜在的 bug 或安全问题

#### 分析 B - 架构设计

分析代码变更中的：

- 与项目现有结构的一致性
- 新增依赖的合理性
- 设计模式的使用
- 关注点分离
- 模块化程度
- API 设计合理性

### 步骤 3: 生成审查报告

将分析结果整合为结构化的审查报告，按严重程度分类：

- **严重 (Critical)**: 必须修复的问题，如安全漏洞、严重 bug
- **重要 (Important)**: 建议修复的问题，如设计缺陷、性能问题
- **建议 (Suggestion)**: 可选的改进建议，如代码风格优化

### 步骤 4: 保存审查摘要

将审查摘要写入文件供通知使用：

```bash
# 写入摘要到文件（用于飞书通知）
cat > /tmp/review-summary.json << 'EOF'
{
  "conclusion": "APPROVE 或 REQUEST_CHANGES 或 COMMENT",
  "summary": "一句话总结审查结果",
  "critical_count": 0,
  "important_count": 0,
  "suggestion_count": 0
}
EOF
```

**重要**: 必须在发布评论前将摘要写入 `/tmp/review-summary.json`，格式为 JSON：

- `conclusion`: 审查结论 (APPROVE/REQUEST_CHANGES/COMMENT)
- `summary`: 一句话总结（不超过 50 字）
- `critical_count`: 严重问题数量
- `important_count`: 重要问题数量
- `suggestion_count`: 建议数量

### 步骤 5: 发布审查评论

使用以下命令发布审查评论：

```bash
gh pr review $PR_NUMBER --repo $REPO --comment --body "审查内容"
```

## 输出格式

审查评论采用以下 Markdown 格式（简体中文）：

```markdown
## 🔍 PR 代码审查报告

### 📋 概要

- **PR 标题**: {title}
- **作者**: {author}
- **变更文件数**: {changedFiles}
- **新增/删除行数**: +{additions} / -{deletions}

### 🎯 审查结论

{APPROVE/REQUEST_CHANGES/COMMENT 及简要说明}

### 🔴 严重问题 (Critical)

{严重问题列表，如无则显示"无"}

### 🟠 重要问题 (Important)

{重要问题列表，如无则显示"无"}

### 🟢 改进建议 (Suggestions)

{改进建议列表，如无则显示"无"}

### 💡 总体评价

{对本次 PR 的整体评价和建议}
```

## 注意事项

1. 所有评论使用简体中文
2. 关注代码质量和架构设计两个维度
3. 给出具体的代码行号和改进建议
4. 对于好的实践也要给予肯定
5. 保持客观、专业的审查态度
