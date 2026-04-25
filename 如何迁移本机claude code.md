# 如何迁移本机 Claude Code

## 这份文档怎么用
这是一份执行版清单，适合你已经决定要迁移，只想快速照着做。

配套文档：
- **当前全局配置总览**：`C:\Users\admin\.claude\README.md`
- **决策手册**：`C:\Users\admin\.claude\如何迁移本机claude code-专家版.md`

## 目标
用最小步骤把当前这台电脑上的 Claude Code 迁到另一台电脑，恢复常用规则、skills、agents、插件配置骨架和个人工作方式，同时不迁缓存、历史状态和敏感密钥。

## 先看一句话结论
最小迁移优先保留：
- `CLAUDE.md`
- `rules`
- `skills`
- `agents`
- 无密钥版 `settings.json`

按需再补：
- `projects\*\memory`
- 经人工复核后的 `.claude\settings.local.json`

不要迁：
- 缓存
- sessions
- history
- plans
- tool-results
- debug / metrics / browser-profiles / shell-snapshots

## 迁移时先分清 3 类配置来源
### 1. 全局主配置
- `C:\Users\admin\.claude\settings.json`
- 用来判断：主配置骨架、显式启用/禁用插件、marketplace 来源、默认 shell、hooks、statusLine 等。

### 2. 本地覆盖配置
- `C:\Users\admin\.claude\.claude\settings.local.json`
- 当前本机该文件存在。
- 它属于本机覆盖层，不应默认当作跨机通用配置直接照搬。

### 3. 插件安装登记
- `C:\Users\admin\.claude\plugins\installed_plugins.json`
- 用来判断哪些插件“装过”。
- 它不等于“当前启用状态”，必须和 `settings.json.enabledPlugins` 分开看。

## 本机当前插件状态
### 显式启用（`enabledPlugins: true`）
- `context7@claude-plugins-official`
- `pyright-lsp@claude-plugins-official`
- `claude-hud@claude-hud`
- `superpowers@claude-plugins-official`
- `github@claude-plugins-official`
- `skill-creator@claude-plugins-official`

### 显式禁用（`enabledPlugins: false`）
- `codex@openai-codex`
- `feature-dev@claude-plugins-official`
- `frontend-design@claude-plugins-official`

### 已安装但未显式声明
- `kotlin-lsp@claude-plugins-official`
- `playwright@claude-plugins-official`
- `claude-md-management@claude-plugins-official`

说明：
- “已安装”以 `installed_plugins.json` 为准。
- “显式启用/禁用”以 `settings.json.enabledPlugins` 为准。
- `claude-md-management` 当前是 `local scope` 插件，绑定特定项目路径；迁移时不要把它当作全局通用插件基线。

## Marketplace 来源清单
恢复插件之前，先确认 `settings.json.extraKnownMarketplaces`。

当前已核实只有这 3 个：

| 市场 ID | 类型 | 地址 |
|---------|------|------|
| `claude-plugins-official` | git | https://github.com/anthropics/claude-plugins-official.git |
| `claude-hud` | github | jarrodwatts/claude-hud |
| `openai-codex` | github | openai/codex-plugin-cc |

迁移时不要继续按照旧文档补 `claude-code-lsps`；是否需要新增来源，应以新机器当时的实际需求为准。

## 迁移涉及的 skills 怎么判断
判断原则只有两类：
- 插件提供的 skill：迁移时恢复对应 marketplace、安装插件、按需启用
- 本地 `C:\Users\admin\.claude\skills` 里的 skill：迁移时复制 `skills` 目录

### 当前仓库可见的本地 skills
- `async-python-patterns`
- `ceeon-best-minds`
- `defuddle`
- `pdf`
- `pdf-converter`
- `python-error-handling`
- `python-testing-patterns`
- `web-access`

说明：
- 文档或历史记录里提到的旧 skill 名，不应直接当作当前仍存在。
- 如果担心遗漏，应在迁移前再次核对 `skills/` 目录现状，而不是照抄旧快照。

### `web-access`
`web-access` 仍是外部仓库来源，迁移后如果要继续使用，单独恢复：

```powershell
git clone https://github.com/eze-is/web-access C:\Users\admin\.claude\skills\web-access
```

运行时要求：启动浏览器时加 `--remote-debugging-port=9222` 参数。更新用 `git pull`。

## 迁移前要准备什么
开始复制文件前，先准备这 5 项：

1. 无密钥版 `settings.json`
   - 只保留跨机仍成立的配置骨架。

2. 当前显式启用插件列表
   - 以 `enabledPlugins: true` 为准。

3. 当前 marketplace 来源
   - 以 `extraKnownMarketplaces` 为准。

4. `env` 键名清单
   - 只记录变量名，不记录变量值。

5. 外部依赖清单
   - 至少确认新机器是否有：PowerShell、bash、Node.js。

## 迁移时必须带什么
### 必须迁移
- `C:\Users\admin\.claude\CLAUDE.md`
- `C:\Users\admin\.claude\rules`
- `C:\Users\admin\.claude\skills`
- `C:\Users\admin\.claude\agents`
- `C:\Users\admin\.claude\settings.json` 的无密钥版本

无密钥版 `settings.json` 只优先保留这些相对稳定的配置：
- `enabledPlugins`
- `extraKnownMarketplaces`
- `permissions` 中可复用部分
- `defaultShell`
- 明确跨机仍成立的通用开关

### 按需迁移
- `C:\Users\admin\.claude\projects\*\memory`
- `C:\Users\admin\.claude\.claude\settings.local.json`

说明：
- `memory` 只迁长期有价值的项目记忆。
- `C:\Users\admin\.claude\.claude\settings.local.json` 默认不跨机照搬；只有当新机器路径结构、权限需求和本地工具链足够接近时才考虑复用。
- `plugins/` 目录本身不作为迁移源；新机器应通过 marketplace + 插件安装 + 启用状态重建。

### 不要迁移
运行态、缓存、个人历史等短期数据：

- `C:\Users\admin\.claude\plugins\`
- `C:\Users\admin\.claude\sessions`
- `C:\Users\admin\.claude\history.jsonl`
- `C:\Users\admin\.claude\browser-profiles`
- `C:\Users\admin\.claude\debug`
- `C:\Users\admin\.claude\metrics`
- `C:\Users\admin\.claude\shell-snapshots`
- `C:\Users\admin\.claude\plans`
- `C:\Users\admin\.claude\paste-cache`
- `C:\Users\admin\.claude\file-history`
- `C:\Users\admin\.claude\backups`
- `C:\Users\admin\.claude\session-env`
- `C:\Users\admin\.claude\tasks`
- `C:\Users\admin\.claude\ide`
- `C:\Users\admin\.claude\homunculus`
- `C:\Users\admin\.claude\rules_archive`
- `C:\Users\admin\.claude\skills_archive`
- 各项目目录下的 `tool-results`

## 敏感信息处理
迁移时，不要传输原始 `settings.json` 或本地覆盖文件里的敏感值。

### `env` 分三类处理
| 类别 | 示例键 | 处理方式 |
|------|--------|---------|
| **密钥/凭据** | `ANTHROPIC_AUTH_TOKEN`、`GITHUB_PERSONAL_ACCESS_TOKEN` | 必须在新机器重新填写，不跨机复制值 |
| **本机代理/模型映射** | `ANTHROPIC_BASE_URL`、`ANTHROPIC_DEFAULT_*_MODEL` | 取决于新机器是否有同一代理/映射关系 |
| **通用开关** | `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`、`CLAUDE_CODE_USE_POWERSHELL_TOOL` | 在目标机仍需要同样行为且环境兼容时可保留 |

### 正确做法
1. 先按三类拆分 `env`，只保留通用开关。
2. 记下密钥和代理的键名，不记录值。
3. 生成无密钥版 `settings.json`。
4. 新机器按键名清单重新填写。
5. 不通过网盘、压缩包、Git 仓库、聊天工具传输含密钥原文件。
6. 如果原文件或 token 曾被外发，迁移后立即轮换。

## 执行顺序
按下面顺序做最稳。

### 第一步：从旧机器导出文件
导出：
- `CLAUDE.md`
- `rules`
- `skills`
- `agents`
- 无密钥版 `settings.json`
- `projects\*\memory`（按需）

### 第二步：在新机器恢复基础文件
将第一步导出的内容（CLAUDE.md、rules、skills、agents、无密钥版 settings.json 及按需的 memory）放到新机器对应的 `~/.claude/` 目录中。

### 第三步：恢复 marketplace 来源
先按 `settings.json.extraKnownMarketplaces` 补齐来源，再安装插件。

### 第四步：恢复插件
恢复顺序：
1. marketplace 来源正确
2. 插件安装成功
3. 按 `enabledPlugins` 恢复显式启用/禁用状态

### 第五步：重新填写 env
只按键名清单在新机器补值，不复制旧值。

### 第六步：最后再处理本地覆盖层
再判断 `C:\Users\admin\.claude\.claude\settings.local.json`、hooks、statusLine、额外 permissions 是否需要迁移。

## 最后核对 4 件事
1. CLAUDE.md / rules / skills / agents 是否就位
2. marketplace 来源是否正确
3. 显式启用插件是否按 `enabledPlugins` 恢复
4. 本地覆盖配置是否经过人工复核，而不是直接照搬
