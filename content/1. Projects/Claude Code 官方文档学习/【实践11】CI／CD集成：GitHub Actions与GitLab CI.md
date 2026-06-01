---
title: "【实践11】CI/CD 集成：GitHub Actions 与 GitLab CI"
publish: true
---

# 【实践11】CI/CD 集成：GitHub Actions 与 GitLab CI

> 把 Claude Code 嵌入 CI/CD 流水线——让 AI 自动处理 Issue、修复 Bug、审查代码，在 merge 之前发现问题。

**参考文档**：
- `code.claude.com/docs/en/github-actions.md` — GitHub Actions
- `code.claude.com/docs/en/gitlab-ci-cd.md` — GitLab CI/CD
- `code.claude.com/docs/en/github-enterprise-server.md` — GitHub Enterprise Server

**先修知识**：[[【8】Hooks自动化工作流]]

**学习目标**：理解CI/CD集成价值、掌握GitHub Actions配置、掌握GitLab CI配置、了解@claude触发、了解Bedrock/VertexAI后端

---

## 1. 为什么把 Claude Code 放进 CI/CD

### 场景：自动修 bug、Issue → MR、@claude 指令触发

将 Claude Code 集成到 CI/CD 流水线中，可以实现：

- **Instant PR/MR creation**: Describe what you need, and Claude creates a complete PR/MR with all necessary changes.
- **Automated code implementation**: Turn issues into working code with a single command.
- **Automated bug fixing**: Claude locates bugs, implements fixes, and updates branches or opens new MRs.
- **Follows your standards**: Claude respects your `CLAUDE.md` guidelines and existing code patterns.
- **Event-driven orchestration**: GitLab/GitHub listens for triggers (e.g., a comment mentioning `@claude`), collects context, and runs Claude Code.
- **Secure by default**: Runs in your own runners with your branch protection and approval rules.

**典型触发链路**：
1. Developer posts `@claude fix the TypeError in the user dashboard component` in an Issue/PR comment.
2. CI picks up the event, checks out the repository.
3. Claude Code analyzes the codebase, implements the fix, and pushes to a branch / opens a PR/MR.
4. Human reviewers see the diff and approve as usual.

> **个人笔记**：
>
>

---

## 2. GitHub Actions 集成

### Setup 步骤

**Quick setup**（推荐，通过 Claude 终端）：
```bash
# 在 Claude 终端中运行
/install-github-app
```
- 需要 repository admin 权限
- GitHub App 请求 Contents、Issues、Pull requests 的读写权限
- 仅适用于直接 Claude API 用户（Bedrock/Vertex 用户需手动配置）

**Manual setup**：
1. Install the Claude GitHub App: https://github.com/apps/claude
   - 所需权限：Contents (Read & Write), Issues (Read & Write), Pull requests (Read & Write)
2. Add `ANTHROPIC_API_KEY` to repository secrets.
3. Copy workflow file into `.github/workflows/`.

### 基本 Workflow 示例

```yaml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
jobs:
  claude:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

自动检测 mode：当提供 `prompt` 时运行 automation mode；当仅用于 issue/PR comment 时运行 interactive mode（响应 `@claude`）。

### Action 参数

| Parameter | Description | Required |
|-----------|-------------|----------|
| `prompt` | Instructions for Claude (plain text or skill name) | No* |
| `claude_args` | CLI arguments passed to Claude Code | No |
| `plugin_marketplaces` | Newline-separated list of plugin marketplace Git URLs | No |
| `plugins` | Newline-separated list of plugin names to install | No |
| `anthropic_api_key` | Claude API key | Yes** |
| `github_token` | GitHub token for API access | No |
| `trigger_phrase` | Custom trigger phrase (default: `@claude`) | No |
| `use_bedrock` | Use Amazon Bedrock instead of Claude API | No |
| `use_vertex` | Use Google Vertex AI instead of Claude API | No |

*Prompt is optional — when omitted for issue/PR comments, Claude responds to trigger phrase.
**Required for direct Claude API, not for Bedrock/Vertex.

### 常用 claude_args

```yaml
claude_args: |
  --max-turns 10
  --model claude-sonnet-4-6
  --mcp-config /path/to/config.json
  --allowedTools "Bash Read Edit Write"
  --debug
```

### Beta 到 v1.0 升级要点

| Old Beta Input | New v1.0 Input |
|---|---|
| `mode` | (Removed - auto-detected) |
| `direct_prompt` | `prompt` |
| `custom_instructions` | `claude_args: --append-system-prompt` |
| `max_turns` | `claude_args: --max-turns` |
| `model` | `claude_args: --model` |

> **个人笔记**：
>
>

---

## 3. GitLab CI/CD 集成

> **Beta 阶段**：目前由 GitLab 维护，功能和 API 可能持续演进。

### Setup 步骤

**Quick setup**：
1. Add masked CI/CD variable `ANTHROPIC_API_KEY` (Settings → CI/CD → Variables).
2. Add a Claude job to `.gitlab-ci.yml`.

### .gitlab-ci.yml 示例

```yaml
stages:
  - ai

claude:
  stage: ai
  image: node:24-alpine3.21
  rules:
    - if: '$CI_PIPELINE_SOURCE == "web"'
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  variables:
    GIT_STRATEGY: fetch
  before_script:
    - apk update
    - apk add --no-cache git curl bash
    - curl -fsSL https://claude.ai/install.sh | bash
  script:
    - /bin/gitlab-mcp-server || true
    - >
      claude
      -p "${AI_FLOW_INPUT:-'Review this MR and implement the requested changes'}"
      --permission-mode acceptEdits
      --allowedTools "Bash Read Edit Write mcp__gitlab"
      --debug
```

### Quick setup vs Manual setup

| Aspect | Quick Setup | Manual Setup |
|--------|-------------|---------------|
| Complexity | Minimal — one job + one variable | Full control, OIDC/WIF config |
| Authentication | Claude API key only | Claude API / Bedrock / Vertex AI |
| Production readiness | Good for prototyping | Recommended for production |
| OIDC support | No | Yes (Bedrock/Vertex) |

**Manual setup 额外步骤**：
1. Configure provider access (API key / Bedrock OIDC / Vertex WIF).
2. Add project credentials for GitLab API (`CI_JOB_TOKEN` or `GITLAB_ACCESS_TOKEN`).
3. (Optional) Enable mention-driven triggers via webhook + pipeline trigger API.

### @claude 触发机制（GitLab 侧）

- 设置 project webhook，监听 "Comments (notes)" 事件。
- 当 comment 包含 `@claude` 时，listener 调用 pipeline trigger API。
- 通过 `AI_FLOW_INPUT` 和 `AI_FLOW_CONTEXT` 变量传递上下文。

> **个人笔记**：
>
>

---

## 4. @claude 指令触发机制

### PR Comment 触发

在 PR 或 Issue 的评论中提及 `@claude`，Claude 会自动：

- 分析上下文（代码、Issue 描述、讨论历史）
- 执行请求的任务（实现功能、修复 Bug、回答实现问题）
- 创建或更新 PR/MR

**典型命令示例**：
```text
@claude implement this feature based on the issue description
@claude how should I implement user authentication for this endpoint?
@claude fix the TypeError in the user dashboard component
@claude suggest a concrete approach to cache the results of this API call
```

### Issue Comment 触发

对于新打开的 Issue，可以在 Issue body 中包含 `@claude`：
```yaml
# GitHub Actions workflow 中的条件判断
if: |
  (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
  (github.event_name == 'issues' && contains(github.event.issue.body, '@claude'))
```

### 自定义触发短语

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    trigger_phrase: "@ai-assistant"  # 默认是 @claude
```

### GitHub Enterprise Server 注意事项

- `/install-github-app` command not supported on GHES — use admin setup flow on claude.ai instead.
- GitHub MCP server does not work with GHES — use `gh` CLI configured for your GHES host.
- Plugin marketplaces on GHES: use full git URL instead of `owner/repo` shorthand.

> **个人笔记**：
>
>

---

## 5. CLAUDE.md 在 CI 环境中的配置要点

### 为什么重要

`CLAUDE.md` defines coding standards, review criteria, project-specific rules, and preferred patterns. Claude reads this file during CI runs and follows your conventions when proposing changes.

### CI 特定的配置建议

```markdown
# CLAUDE.md — CI/CD Environment

## Coding Standards
- Use TypeScript strict mode
- Follow the existing code style in the repository
- All new code must include unit tests

## CI-Specific Directives
- When fixing bugs, always add a regression test
- When implementing features, follow the existing project structure
- Do not modify configuration files (CI/CD configs, etc.)
- Use the project's established patterns for error handling

## Security Requirements
- Never commit hardcoded credentials
- Always validate user input
- Follow OWASP Top 10 guidelines

## Review Criteria
- Check for potential regressions
- Verify test coverage meets threshold (80%+)
- Ensure documentation is updated
```

### 在 Workflow 中追加指令

```yaml
# GitHub Actions
- uses: anthropics/claude-code-action@v1
  with:
    claude_args: >
      --append-system-prompt "Focus on security review. 
      Check for OWASP Top 10 vulnerabilities."
```

### 最佳实践

- **Keep it concise**: Claude reads `CLAUDE.md` every run — avoid unnecessary verbosity.
- **Use separate prompts for different jobs**: Review job vs. implement job may need different instructions.
- **Place at repository root**: Claude finds it automatically.
- **Version control it**: Changes to `CLAUDE.md` go through PR review like any other file.

> **个人笔记**：
>
>

---

## 6. Bedrock / Vertex AI 作为 CI 后端

### 为什么需要 3P 后端

- **Data residency**: Keep data within your cloud region.
- **Existing cloud agreements**: Leverage committed cloud spend.
- **Enterprise compliance**: Meet regulatory requirements.
- **Billing consolidation**: Use existing cloud billing.

### Amazon Bedrock

**认证方式：OIDC（OpenID Connect）**

```yaml
# GitHub Actions — Bedrock Workflow
permissions:
  contents: write
  pull-requests: write
  issues: write
  id-token: write

jobs:
  claude-pr:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Generate GitHub App token
        uses: actions/create-github-app-token@v2
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.APP_PRIVATE_KEY }}
      - name: Configure AWS Credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
          aws-region: us-west-2
      - uses: anthropics/claude-code-action@v1
        with:
          github_token: ${{ steps.app-token.outputs.token }}
          use_bedrock: "true"
          claude_args: '--model us.anthropic.claude-sonnet-4-6 --max-turns 10'
```

**GitLab CI 下的 Bedrock OIDC**：

```yaml
claude-bedrock:
  stage: ai
  image: node:24-alpine3.21
  before_script:
    - apk add --no-cache bash curl jq git python3 py3-pip
    - pip install --no-cache-dir awscli
    - curl -fsSL https://claude.ai/install.sh | bash
    # Exchange GitLab OIDC token for AWS credentials
    - export AWS_WEB_IDENTITY_TOKEN_FILE="${CI_JOB_JWT_FILE:-/tmp/oidc_token}"
    - if [ -n "${CI_JOB_JWT_V2}" ]; then printf "%s" "$CI_JOB_JWT_V2" > "$AWS_WEB_IDENTITY_TOKEN_FILE"; fi
    - aws sts assume-role-with-web-identity
      --role-arn "$AWS_ROLE_TO_ASSUME"
      --role-session-name "gitlab-claude-$(date +%s)"
      --web-identity-token "file://$AWS_WEB_IDENTITY_TOKEN_FILE"
      --duration-seconds 3600 > /tmp/aws_creds.json
    - export AWS_ACCESS_KEY_ID="$(jq -r .Credentials.AccessKeyId /tmp/aws_creds.json)"
    - export AWS_SECRET_ACCESS_KEY="$(jq -r .Credentials.SecretAccessKey /tmp/aws_creds.json)"
    - export AWS_SESSION_TOKEN="$(jq -r .Credentials.SessionToken /tmp/aws_creds.json)"
  script:
    - claude -p "${AI_FLOW_INPUT:-'Implement changes and open an MR'}"
      --permission-mode acceptEdits
      --allowedTools "Bash Read Edit Write mcp__gitlab"
  variables:
    AWS_REGION: "us-west-2"
```

### Google Vertex AI

**认证方式：Workload Identity Federation (WIF)**

```yaml
# GitHub Actions — Vertex AI Workflow
- name: Authenticate to Google Cloud
  uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
    service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
- uses: anthropics/claude-code-action@v1
  with:
    github_token: ${{ steps.app-token.outputs.token }}
    trigger_phrase: "@claude"
    use_vertex: "true"
    claude_args: '--model claude-sonnet-4-5@20250929 --max-turns 10'
  env:
    CLOUD_ML_REGION: us-east5
```

**所需 Secrets 对比**：

| Provider | Required Secrets |
|----------|------------------|
| Claude API (Direct) | `ANTHROPIC_API_KEY` |
| Amazon Bedrock | `AWS_ROLE_TO_ASSUME`, `APP_ID`, `APP_PRIVATE_KEY` |
| Google Vertex AI | `GCP_WORKLOAD_IDENTITY_PROVIDER`, `GCP_SERVICE_ACCOUNT`, `APP_ID`, `APP_PRIVATE_KEY` |

> **个人笔记**：
>
>

---

## 7. 安全与成本控制

### 最小权限原则

- **Never commit API keys**: Always use GitHub Secrets or GitLab CI/CD masked variables.
- **OIDC over static keys**: IAM roles and Workload Identity Federation eliminate long-lived credentials.
- **Restrict job permissions**: Limit what tokens can do — `CI_JOB_TOKEN` scoped to specific operations.
- **Review AI-generated code**: Claude's changes flow through PR/MR — reviewers see every diff.
- **Branch protection**: Apply the same approval rules to AI-generated code.

### Spend Cap 与成本控制

**API 成本**：
- Each Claude interaction consumes tokens based on prompt and response size.
- Token usage varies by task complexity and codebase size.
- Pricing details: anthropic.com/pricing

**CI Runner 成本**：
- GitHub Actions: consumes your GitHub Actions minutes.
- GitLab CI: consumes your GitLab runner compute minutes.

**成本优化策略**：

| Strategy | Implementation |
|----------|----------------|
| 精准指令 | Use specific `@claude` commands to reduce unnecessary API calls |
| 限制对话轮次 | `claude_args: --max-turns 5` |
| 工作流超时 | Set workflow-level timeouts to avoid runaway jobs |
| 并发控制 | Use GitHub concurrency controls / GitLab resource groups |
| 选用合适模型 | Sonnet for daily tasks, Opus only for complex analysis |
| Cache dependencies | Cache npm/package installs in runners where possible |

### Security Architecture

```
┌─────────────────────────────────────────────────┐
│                  CI/CD Pipeline                   │
│  ┌─────────────┐    ┌──────────────────────┐    │
│  │ Event       │───>│ Sandboxed Container  │    │
│  │ (comment/   │    │  - Isolated network  │    │
│  │  issue/     │    │  - Workspace-scoped  │    │
│  │  schedule)  │    │    permissions       │    │
│  └─────────────┘    │  - No static creds   │    │
│                     └──────────┬───────────┘    │
│                                │                 │
│                     ┌──────────▼───────────┐    │
│                     │  Claude Code         │    │
│                     │  → Analyzes code     │    │
│                     │  → Makes changes     │    │
│                     │  → Opens PR/MR       │    │
│                     └──────────┬───────────┘    │
│                                │                 │
│                     ┌──────────▼───────────┐    │
│                     │  Human Review        │    │
│                     │  → Branch protection │    │
│                     │  → Approval rules    │    │
│                     │  → Merge             │    │
│                     └──────────────────────┘    │
└─────────────────────────────────────────────────┘
```

### GitHub Enterprise Server 安全补充

- GHES instance must be reachable from Anthropic infrastructure — firewall must allowlist Anthropic API IP addresses.
- Admin-managed GitHub App with pre-configured permissions.
- Self-signed CA certificates supported in admin setup.

> **个人笔记**：
>
>

---

## 总结对比

| Feature | GitHub Actions | GitLab CI/CD |
|---------|---------------|--------------|
| Status | GA (v1.0) | Beta |
| Setup | `/install-github-app` or manual | Add job + variable |
| Trigger | `@claude` in comments (auto-detect) | Webhook + pipeline API |
| Authentication | API key / Bedrock OIDC / Vertex WIF | API key / Bedrock OIDC / Vertex WIF |
| Action/Job | `anthropics/claude-code-action@v1` | `node:24-alpine3.21` + install script |
| CLI args | `claude_args` input | Direct `claude` command |
| Max turns | `--max-turns` in claude_args | `--max-turns` flag |
| Plugin support | `plugin_marketplaces` + `plugins` inputs | MCP server + direct CLI |
| Enterprise | GHES supported (admin setup) | Not specifically mentioned |

---

## 参考链接

- [Claude Code GitHub Actions Docs](https://code.claude.com/docs/en/github-actions.md)
- [Claude Code GitLab CI/CD Docs](https://code.claude.com/docs/en/gitlab-ci-cd.md)
- [Claude Code with GitHub Enterprise Server](https://code.claude.com/docs/en/github-enterprise-server.md)
- [Claude Code Action Repository](https://github.com/anthropics/claude-code-action)
- [Claude Pricing](https://www.anthropic.com/pricing)
