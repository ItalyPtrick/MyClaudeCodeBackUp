# Claude Code 全局配置备份

> 我的 Claude Code 用户级配置备份，仅供个人参考和对外展示工作流。
> 所有敏感信息（API key、token、历史会话）均已通过 `.gitignore` 排除。

---

## 板块一：配置分层（按响应顺序）

Claude Code 接到请求时的生效链路：**系统提示 → 行为规范 → 任务子代理 → 工作流 skill → 插件扩展**。

### 1. `CLAUDE.md` — 全局系统提示

每次会话注入的约束层，优先级最高。

- **环境**：Windows 11 / PowerShell，优先 Python（Miniconda），中文回复
- **动手前先想清楚**：不假设、不藏困惑、追根因
- **简洁优先**：最少代码解决问题，不做投机性扩展
- **外科手术式改动**：只碰必须碰的，不顺手优化
- **目标驱动执行**：先定义验收标准，再执行，再验证
- **`<output_contract>` / `<verbosity_controls>`**：强制回答长度上限与禁止的表达模式

### 2. `rules/` — 行为规范层

| 文件 | 内容 |
|------|------|
| `rules/common/coding-style.md` | 不可变性优先、文件大小控制、错误处理 |
| `rules/common/testing.md` | TDD 工作流、80% 覆盖率要求 |
| `rules/common/git-workflow.md` | commit 格式、PR 流程 |
| `rules/common/security.md` | 提交前安全检查清单、密钥管理 |
| `rules/common/agents.md` | Agent 编排规则、何时并行调用 |

> `rules/README.md` 描述了多语言扩展层（`typescript/ python/ golang/ swift/`），当前仓库尚未落地，仅 `common/` 生效。

### 3. `agents/` — 任务型子代理

由 Claude 自动调用执行有限范围任务：

| Agent | 用途 |
|-------|------|
| `planner` | 复杂功能的实施规划 |
| `code-reviewer` | 代码写完后自动 review |
| `tdd-guide` | 新功能/修 bug 时强制 TDD 流程 |
| `build-error-resolver` | 构建失败时修复错误 |
| `refactor-cleaner` | 死代码清理 |
| `doc-updater` | 文档同步更新 |

### 4. `skills/` — 工作流指令库

按需调用的深度操作指南。仓库中同时包含个人添加的 skill 和来自插件市场同步的 skill。

#### 4.1 个人添加 / 核心

| Skill | 用途 |
|-------|------|
| `ceeon-best-minds` | 模拟顶级专家多视角思考 |
| `web-access` ⁎ | 联网操作统一入口（CDP 浏览器调度） |
| `defuddle` | 网页抓取转干净 Markdown |
| `obsidian-cli` | Obsidian 仓库 CLI 操作 |
| `obsidian-markdown` | Obsidian Flavored Markdown 语法 |
| `obsidian-bases` | Obsidian `.base` 视图/公式 |
| `json-canvas` | `.canvas` 文件节点/边/分组 |
| `sde-notes` | 工程笔记沉淀规范 |
| `simple` | 简单任务快捷模板 |

> ⁎ `web-access` 不随本仓库同步：通过 `git clone https://github.com/eze-is/web-access C:\Users\admin\.claude\skills\web-access` 独立安装，`.gitignore` 显式排除（见本文底部"不在此备份中的内容"）。更新用 `git pull`。运行时需以 `--remote-debugging-port=9222` 参数启动浏览器（CentBrowser / Chrome 均可），脚本通过该端口 fallback 探测 CDP。

#### 4.2 从插件市场同步（superpowers / everything-claude-code）

**规划与执行**
- `writing-plans`、`executing-plans`、`planning-with-files`、`brainstorming`

**调试与验证**
- `systematic-debugging`、`verification-before-completion`、`test-driven-development`

**Skill 元工具**
- `skill-creator`、`find-skills`、`using-superpowers`、`dispatching-parallel-agents`

**AI / API 开发**
- `claude-api`、`rag-implementation`、`embedding-strategies`、`fastapi-templates`、`api-design-principles`

**Python 专项**
- `async-python-patterns`、`python-error-handling`、`python-testing-patterns`

**文件 / 自动化**
- `pdf`、`docx`、`xlsx`、`agent-browser`、`webapp-testing`

### 5. `plugins/` — 插件市场扩展

> `plugins/` 目录整体不纳入 git（Claude CLI 运行时产物）。下文清单为快照式记录，权威源是 `settings.json.enabledPlugins` 和 `extraKnownMarketplaces`。

**市场源**

| 市场 | 地址 |
|------|------|
| `claude-plugins-official` | https://github.com/anthropics/claude-plugins-official.git |
| `claude-code-lsps` | https://github.com/Piebald-AI/claude-code-lsps.git |
| `claude-hud` | https://github.com/jarrodwatts/claude-hud |
| `openai-codex` | https://github.com/openai/codex-plugin-cc |

**已安装插件**

状态取自 `settings.json.enabledPlugins`：`启用` = `true`，`禁用` = `false`，`默认` = 未在该字段中出现。

| 插件 | 市场 | 状态 | 用途 |
|------|------|------|------|
| `commit-commands` | claude-plugins-official | 启用 | git commit / push / PR 一键操作 |
| `context7` | claude-plugins-official | 启用 | 实时拉取最新库文档 |
| `code-review` | claude-plugins-official | 启用 | PR 代码 review |
| `github` | claude-plugins-official | 启用 | GitHub API 操作 |
| `serena` | claude-plugins-official | 启用 | 语义代码分析（LSP 级别） |
| `superpowers` | claude-plugins-official | 启用 | skill 体系核心插件 |
| `pyright-lsp` | claude-plugins-official | 启用 | Python 类型检查 LSP |
| `pyright` | claude-code-lsps | 启用 | 社区 Pyright 封装 |
| `powershell-editor-services` | claude-code-lsps | 启用 | PowerShell LSP |
| `claude-hud` | claude-hud | 启用 | 会话状态 HUD |
| `codex` | openai-codex | 禁用 | OpenAI Codex 集成 |
| `feature-dev` | claude-plugins-official | 禁用 | 功能开发引导流程 |
| `frontend-design` | claude-plugins-official | 禁用 | 前端设计指导 |
| `playwright` | claude-plugins-official | 默认 | 浏览器自动化测试 |
| `kotlin-lsp` | claude-plugins-official | 默认 | Kotlin LSP |

---

## 板块二：如何复用

**提升工作质量** — 从 `rules/common/` 拷贝规范到自己的 `~/.claude/rules/`，重点看 `testing.md` 和 `security.md`。

**使用 skill 体系**
1. 安装 `superpowers`：`/install-plugin superpowers@claude-plugins-official`
2. 按需拷贝 `skills/` 下目录到你的 `~/.claude/skills/`
3. Claude 会依任务类型自动触发，或在对话中直接引用 skill 名

**使用 Agent 编排** — 拷贝 `agents/*.md` 到 `~/.claude/agents/`，配合 `rules/common/agents.md` 了解调用规则。

**同步插件** — 参考 `settings.json.enabledPlugins` 的插件 ID 和 `extraKnownMarketplaces` 的市场源，在新机器用 `claude plugin marketplace add` / `claude plugin install` 逐个恢复。

**迁移到新机器** — 仓库内附两份迁移手册：
- `如何迁移本机claude code.md` — 执行版清单
- `如何迁移本机claude code-专家版.md` — 决策版（四层模型、env 分类、路径耦合）

### 不在此备份中的内容

`.gitignore` 采用 `*` 默认全排除，仅白名单放行：目录 `agents/`、`rules/`、`skills/`（`skills/web-access/` 除外）；根文件 `CLAUDE.md`、`README.md`、`无密钥版settings.json`、`如何迁移本机claude code.md`、`如何迁移本机claude code-专家版.md`。以下明确被排除：

| 排除项 | 原因 |
|--------|------|
| `settings.json` | 含 API token 等密钥 |
| `config.json` | 本地路径与配置 |
| `history.jsonl` | 输入历史 |
| `sessions/` `projects/` `paste-cache/` `file-history/` | 会话与操作历史 |
| `backups/` `cache/` `debug/` `metrics/` `shell-snapshots/` | 临时与缓存数据 |
| `plans/` | 项目特定的临时实施计划 |
| `plugins/` | Claude CLI 运行时目录：插件注册、marketplace gitlink、transcript 缓存等，完全交还 CLI 自管；跨机通过 `settings.json.enabledPlugins` + `extraKnownMarketplaces` 驱动重装 |
| `skills/web-access/` | 外部仓库（`eze-is/web-access`），本地独立 `git clone` 管理 |
