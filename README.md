# CR-template

基于 Claude AI 的自动化代码审查模板，支持 PR Review 和 Commit Review，并通过飞书 Webhook 发送通知。

## 功能特性

| 功能 | 说明 |
|-----|------|
| PR Review | PR 创建/更新时自动审查代码质量和架构设计 |
| Commit Review | v* 分支 push 时自动审查最新 commit |
| 飞书通知 | 审查完成后发送卡片消息到飞书群 |
| 中文输出 | 所有审查报告和通知使用简体中文 |

## 快速开始

### 1. Fork 或复制此模板

将此仓库作为模板创建新仓库，或直接复制以下文件到你的项目：

```
.github/workflows/
├── pr-review.yml       # PR 审查工作流
└── commit-review.yml   # Commit 审查工作流

.claude/commands/
├── review-pr.md        # PR 审查命令
└── review-commit.md    # Commit 审查命令
```

### 2. 配置 GitHub Secrets

在仓库 **Settings → Secrets and variables → Actions** 中添加：

| Secret 名称 | 必填 | 说明 |
|------------|:----:|------|
| `ANTHROPIC_API_KEY` | ✅ | Anthropic API 密钥 |
| `ANTHROPIC_BASE_URL` | ❌ | API 基础 URL（使用代理时需要） |
| `FEISHU_WEBHOOK_TOKEN` | ✅ | 飞书机器人 Webhook Token |

### 3. 获取飞书 Webhook Token

1. 在飞书群中添加自定义机器人
2. 复制 Webhook URL：`https://open.feishu.cn/open-apis/bot/v2/hook/{TOKEN}`
3. 将 `{TOKEN}` 部分添加到 GitHub Secrets 的 `FEISHU_WEBHOOK_TOKEN`

### 4. 完成

配置完成后，代码审查将自动运行。

## 触发条件

### PR Review

当 Pull Request 满足以下条件时触发：

| 事件 | 说明 |
|-----|------|
| `opened` | PR 被创建 |
| `synchronize` | PR 有新的提交 |
| `ready_for_review` | PR 从草稿变为就绪 |
| `reopened` | PR 被重新打开 |

> **注意**: 草稿 PR 不会触发审查

### Commit Review

当 push 到以下分支时触发：

- `v*` - 所有以 `v` 开头的分支（如 `v1.0`、`v2.0-beta`）

## 审查维度

### PR Review

**代码质量**
- 代码风格一致性
- 代码复杂度
- 重复代码检测
- 命名规范
- 错误处理
- 潜在 bug 和安全问题

**架构设计**
- 项目结构一致性
- 依赖合理性分析
- 设计模式使用
- 关注点分离
- 模块化程度
- API 设计评估

### Commit Review

- Commit 信息是否清晰
- 代码变更是否合理
- 是否有明显问题

## 审查报告示例

### PR Review 报告

```markdown
## 🔍 PR 代码审查报告

### 📋 概要
- **PR 标题**: feat: 添加用户认证模块
- **作者**: developer
- **变更文件数**: 5
- **新增/删除行数**: +120 / -15

### 🎯 审查结论
✅ APPROVE - 代码质量良好，建议合并

### 🔴 严重问题 (Critical)
无

### 🟠 重要问题 (Important)
1. `src/auth.ts:45` - 密码哈希使用了不推荐的 MD5，建议使用 bcrypt

### 🟢 改进建议 (Suggestions)
1. 考虑添加更多的错误处理边界情况
2. 建议为公共 API 添加 JSDoc 注释

### 💡 总体评价
本次 PR 实现了用户认证的核心功能，代码结构清晰...
```

### Commit Review 报告

```markdown
## 📝 Commit 审查 - v1.0

**Commit**: `abc1234`
**信息**: feat: 添加登录功能

### 变更概要
- src/login.ts (+50)
- tests/login.test.ts (+30)

### 审查结果
✅ PASS - 代码变更合理，无明显问题
```

## 飞书通知

审查完成后会发送飞书卡片消息：

### PR Review 通知

```
┌──────────────────────────────────┐
│ ✅ PR Review - APPROVE           │ (绿色卡片)
├──────────────────────────────────┤
│ 仓库: owner/repo    PR: #123     │
│ 标题: feat: 新功能                │
│ 审查结果: 代码质量良好，建议合并    │
│ 🔴 严重: 0  🟠 重要: 1  🟢 建议: 3 │
│ [查看 PR]                        │
└──────────────────────────────────┘
```

### Commit Review 通知

```
┌──────────────────────────────────┐
│ ✅ Commit Review - PASS          │ (绿色卡片)
├──────────────────────────────────┤
│ 仓库: owner/repo   分支: v1.0    │
│ Commit: abc1234                  │
│ 审查结果: 代码变更合理，无明显问题  │
│ [查看 Commit]                    │
└──────────────────────────────────┘
```

### 卡片颜色说明

| 审查结论 | 卡片颜色 |
|---------|---------|
| APPROVE / PASS | 🟢 绿色 |
| REQUEST_CHANGES / FAIL | 🔴 红色 |
| WARN | 🟠 橙色 |
| COMMENT / 其他 | 🔵 蓝色 |

## 自定义配置

### 修改触发分支

编辑 `.github/workflows/commit-review.yml`：

```yaml
on:
  push:
    branches:
      - 'v*'        # 所有 v 开头的分支
      - 'release/*' # 添加 release 分支
      - 'main'      # 添加 main 分支
```

### 修改审查维度

编辑 `.claude/commands/review-pr.md` 或 `review-commit.md` 中的审查流程。

## 文件结构

```
.
├── .github/
│   └── workflows/
│       ├── pr-review.yml        # PR 审查工作流
│       └── commit-review.yml    # Commit 审查工作流
├── .claude/
│   └── commands/
│       ├── review-pr.md         # PR 审查命令定义
│       └── review-commit.md     # Commit 审查命令定义
└── README.md
```

## License

MIT
