---
description: "简单审查分支的最新 commit"
argument-hint: "[REPO: owner/repo SHA: commit_sha BRANCH: branch_name]"
---

# /review-commit

对分支的最新 commit 进行简单审查。

## 输入参数

- `REPO`: 仓库名称
- `SHA`: Commit SHA
- `BRANCH`: 分支名称

## 审查流程

### 步骤 1: 获取 Commit 信息

```bash
# 获取 commit 详情
git show --stat $SHA

# 获取 commit diff
git show $SHA
```

### 步骤 2: 快速分析

检查以下内容：
- commit 信息是否清晰
- 代码变更是否合理
- 是否有明显问题

### 步骤 3: 保存审查摘要

将审查摘要写入文件供通知使用：

```bash
# 写入摘要到文件（用于飞书通知）
cat > /tmp/review-summary.json << 'EOF'
{
  "conclusion": "PASS 或 WARN 或 FAIL",
  "summary": "一句话总结审查结果"
}
EOF
```

**重要**: 必须将摘要写入 `/tmp/review-summary.json`，格式为 JSON：
- `conclusion`: 审查结论 (PASS/WARN/FAIL)
- `summary`: 一句话总结（不超过 50 字）

### 步骤 4: 输出简要报告

## 输出格式

```markdown
## 📝 Commit 审查 - {BRANCH}

**Commit**: `{SHA}`
**信息**: {commit message}

### 变更概要
{变更文件列表}

### 审查结果
✅ / ⚠️ / ❌ {简要评价}

{如有问题，列出 1-3 个要点}
```

## 注意事项

1. 保持简洁，不需要详细分析
2. 只关注明显问题
3. 使用简体中文
