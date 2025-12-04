# CR-template

基于 Claude AI 的自动化 PR 代码审查模板，通过飞书 Webhook 发送通知。

## 功能特性

| 功能 | 说明 |
|-----|------|
| PR Review | PR 创建/更新时自动审查代码质量和架构设计 |
| 飞书通知 | 审查完成后发送卡片消息到飞书群 |
| 中文输出 | 所有审查报告和通知使用简体中文 |
| Sticky Comment | PR 多次审查会更新同一条评论 |

## 快速开始

### 1. Fork 或复制此模板

将此仓库作为模板创建新仓库，或直接复制以下文件到你的项目：

```
.github/workflows/
└── pr-review.yml       # PR 审查工作流
```

### 2. 配置 GitHub Secrets

在仓库 **Settings → Secrets and variables → Actions** 中添加：

| Secret 名称 | 必填 | 说明 |
|------------|:----:|------|
| `ANTHROPIC_API_KEY` | ✅ | Anthropic API 密钥 |
| `ANTHROPIC_BASE_URL` | ❌ | API 基础 URL（使用代理时需要） |
| `CUSTOM_GITHUB_TOKEN` | ❌ | 自定义 GitHub Token（默认使用 GITHUB_TOKEN） |
| `FEISHU_WEBHOOK_TOKEN` | ✅ | 飞书机器人 Webhook Token |

### 3. 获取飞书 Webhook Token

1. 在飞书群中添加自定义机器人
2. 复制 Webhook URL：`https://open.feishu.cn/open-apis/bot/v2/hook/{TOKEN}`
3. 将 `{TOKEN}` 部分添加到 GitHub Secrets 的 `FEISHU_WEBHOOK_TOKEN`

### 4. 完成

配置完成后，代码审查将自动运行。

## 触发条件

当 Pull Request 满足以下条件时触发：

| 事件 | 说明 |
|-----|------|
| `opened` | PR 被创建 |
| `synchronize` | PR 有新的提交 |
| `ready_for_review` | PR 从草稿变为就绪 |
| `reopened` | PR 被重新打开 |

> **注意**: 草稿 PR 不会触发审查

## 审查维度

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

## 审查报告示例

```markdown
## 🔍 PR 代码审查报告

### 📋 概要
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

## 飞书通知

审查完成后会发送飞书卡片消息：

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

### 卡片颜色说明

| 审查结论 | 卡片颜色 |
|---------|---------|
| APPROVE | 🟢 绿色 |
| REQUEST_CHANGES | 🔴 红色 |
| COMMENT / 其他 | 🔵 蓝色 |

## 自定义配置

### 修改审查 Prompt

编辑 workflow 文件中的 `prompt` 字段：

```yaml
- name: PR Review with Claude
  uses: anthropics/claude-code-action@v1
  with:
    prompt: |
      你是一个专业的代码审查助手...
      # 自定义审查要求
```

### 修改结构化输出

编辑 `claude_args` 中的 `--json-schema`：

```yaml
claude_args: |
  --json-schema '{"type":"object","properties":{...}}'
```

## 文件结构

```
.
├── .github/
│   └── workflows/
│       └── pr-review.yml        # PR 审查工作流
└── README.md
```

## 技术说明

- 使用 [claude-code-action](https://github.com/anthropics/claude-code-action) v1
- 通过 `--json-schema` 获取结构化输出用于飞书通知
- 使用 `use_sticky_comment` 合并多次评论

## License

MIT
