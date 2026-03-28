# Claude Code 全局配置备份

> 我的 Claude Code 用户级配置备份，仅供个人参考和对外展示工作流。
> 所有敏感信息（API key、token）均已排除在外。

---

## 板块一：我配置了什么

### CLAUDE.md（全局系统提示）

- `CLAUDE.md` — 通用规范（当前生效）

主要内容：环境信息、交互原则、安全底线（禁止 API key 进 Git）。

---

### Rules（行为规范层）

位于 `rules/common/`，对所有项目生效：

| 文件 | 内容 |
|------|------|
| `coding-style.md` | 不可变性优先、文件大小控制、错误处理 |
| `testing.md` | TDD 工作流、80% 覆盖率要求 |
| `git-workflow.md` | commit 格式、PR 流程 |
| `security.md` | 提交前安全检查清单、密钥管理 |
| `agents.md` | Agent 编排规则、何时并行调用 |

---

### Agents（任务型子代理）

位于 `agents/`，由 Claude 自动调用执行有限范围任务：

| Agent | 用途 |
|-------|------|
| `planner` | 复杂功能的实施规划 |
| `code-reviewer` | 代码写完后自动 review |
| `tdd-guide` | 新功能/修 bug 时强制 TDD 流程 |
| `build-error-resolver` | 构建失败时修复错误 |
| `refactor-cleaner` | 死代码清理 |
| `doc-updater` | 文档同步更新 |

---

### Skills（工作流指令库）

位于 `skills/`，按需调用的深度操作指南：

**AI 开发**
- `claude-api` — Anthropic SDK 集成模式
- `rag-implementation` — RAG 系统构建
- `embedding-strategies` — 向量嵌入选型
- `fastapi-templates` — FastAPI 生产级模板

**工程流程**
- `brainstorming` — 创意转设计稿（有 hard gate）
- `writing-plans` — 实施计划写入文件
- `executing-plans` — 执行已有计划
- `systematic-debugging` — 结构化 debug 流程
- `test-driven-development` — TDD 强制工作流
- `verification-before-completion` — 完成前验证清单
- `skill-creator` — 从历史 session 提取 skill

**内容与文件处理**
- `pdf`, `docx`, `xlsx` — 各格式文件读写
- `obsidian-cli`, `obsidian-markdown`, `obsidian-bases` — Obsidian 笔记操作
- `defuddle` — 网页抓取转 Markdown
- `web-access` — 联网操作统一入口

**Python 专项**
- `async-python-patterns` — asyncio 并发模式
- `python-error-handling` — 异常处理规范
- `python-testing-patterns` — pytest 测试策略

**其他**
- `agent-browser` — 浏览器自动化
- `api-design-principles` — REST/GraphQL 设计原则
- `json-canvas`, `sde-notes`, `simple` 等

---

### Plugins（MCP 插件）

通过 Claude Code 插件市场安装，配置见 `plugins/installed_plugins.json`：

| 插件 | 来源 | 用途 |
|------|------|------|
| `commit-commands` | claude-plugins-official | git commit / push / PR 一键操作 |
| `frontend-design` | claude-plugins-official | 前端设计指导 |
| `context7` | claude-plugins-official | 实时拉取最新库文档 |
| `feature-dev` | claude-plugins-official | 功能开发引导流程 |
| `code-review` | claude-plugins-official | PR 代码 review |
| `github` | claude-plugins-official | GitHub API 操作 |
| `superpowers` | claude-plugins-official | skill 体系核心插件 |
| `serena` | claude-plugins-official | 语义代码分析（LSP 级别） |
| `everything-claude-code` | everything-claude-code | 大量工程 skill 集合 |
| `pyright-lsp` | claude-plugins-official | Python 类型检查 LSP |

插件市场来源：
- `https://github.com/anthropics/claude-plugins-official.git`
- `https://github.com/affaan-m/everything-claude-code.git`
- `https://github.com/Piebald-AI/claude-code-lsps.git`

---

## 板块二：如何参考这份配置

### 对你有价值的部分

**如果你想提升 Claude Code 工作质量：**
- 从 `rules/common/` 开始，把规范加进你自己的 `CLAUDE.md`
- 重点看 `testing.md`（TDD 要求）和 `security.md`（提交前清单）

**如果你想使用 skill 体系：**
1. 安装 `superpowers` 插件：在 Claude Code 中运行 `/install-plugin superpowers@claude-plugins-official`
2. 安装 `everything-claude-code` 插件获取更多 skill
3. 把 `skills/` 下你感兴趣的目录复制到你的 `~/.claude/skills/`
4. 触发方式：Claude 会根据任务类型自动判断是否调用 skill，或你在对话中直接引用

**如果你想使用 Agent 编排：**
- 把 `agents/` 下的 `.md` 文件复制到你的 `~/.claude/agents/`
- 配合 `rules/common/agents.md` 了解调用规则

**如果你只想安装相同的插件：**
- 参考 `plugins/installed_plugins.json` 里的插件 ID
- 在 Claude Code 中逐个安装

### 不在此备份中的内容

| 排除项 | 原因 |
|--------|------|
| `settings.json` | 含 API token 等密钥 |
| `settings.local.json` | 含个人本地路径 |
| `plans/` | 临时实施计划，项目特定，自动生成 |
| `plugins/cache/` | 插件二进制缓存，可自动重新下载 |
| `sessions/`, `projects/` 等 | 历史会话数据，属个人隐私 |
