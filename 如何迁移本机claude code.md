# 如何迁移本机 Claude Code

## 这份文档怎么用
这是一份执行版清单，适合你已经决定要迁移，只想快速照着做。

配套文档：
- **当前环境快照**（插件 / skills / rules / agents 等资产全景）：`C:\Users\admin\.claude\README.md`
- **决策手册**（为什么这样迁、哪些配置本质上不能直接搬、跨机最容易坏在哪）：`C:\Users\admin\Desktop\如何迁移本机claude code-专家版.md`

## 目标
用最小步骤把当前这台电脑上的 Claude Code 迁到另一台电脑，恢复常用插件、skills、rules、agents 和个人工作方式，同时不迁缓存、历史状态和敏感密钥。

## 先看一句话结论
最小迁移优先保留：
- `CLAUDE.md`
- `rules`
- `skills`
- `agents`
- 无密钥版 `settings.json`

按需再补：
- `projects\*\memory`
- 经人工复核后的 `settings.local.json`

不要迁：
- 缓存
- sessions
- history
- plans
- tool-results
- debug / metrics / browser-profiles / shell-snapshots

## 本机当前插件状态
以 `C:\Users\admin\.claude\settings.json` 的 `enabledPlugins` 字段为准（`plugins/` 目录整体不纳入 git，不应作为迁移源）。

> 旧机器可在迁移前执行 `claude plugin list > plugins-snapshot.txt` 留一份快照，作为新机器逐个核对的参考。

### 显式启用（`enabledPlugins: true`）
- `code-review@claude-plugins-official`
- `commit-commands@claude-plugins-official`
- `context7@claude-plugins-official`
- `github@claude-plugins-official`
- `powershell-editor-services@claude-code-lsps`
- `serena@claude-plugins-official`
- `pyright-lsp@claude-plugins-official`
- `claude-hud@claude-hud`
- `superpowers@claude-plugins-official`
- `pyright@claude-code-lsps`

### 显式禁用（`enabledPlugins: false`）
- `codex@openai-codex`
- `feature-dev@claude-plugins-official`
- `frontend-design@claude-plugins-official`

### 已安装但未在 `enabledPlugins` 中出现（默认状态）
- `playwright@claude-plugins-official`
- `kotlin-lsp@claude-plugins-official`

说明：
- "默认状态"不等于"已禁用"。迁移时若希望保持现状，这两个插件的 key 可以继续不写进新机器的 `enabledPlugins`。
- `everything-claude-code@everything-claude-code` 曾经装过，当前已卸载，不属于迁移目标。

## Marketplace 来源清单
恢复插件之前必须先注入这四个市场源（见 `settings.json.extraKnownMarketplaces`）：

| 市场 ID | 类型 | 地址 |
|---------|------|------|
| `claude-plugins-official` | git | https://github.com/anthropics/claude-plugins-official.git |
| `claude-code-lsps` | git | https://github.com/Piebald-AI/claude-code-lsps.git |
| `claude-hud` | github | jarrodwatts/claude-hud |
| `openai-codex` | github | openai/codex-plugin-cc |

## 迁移涉及的 skill 全量清单
这部分列的是你当前环境里，迁移后如果还想继续使用，就需要一并恢复来源的 skill。判断原则只有两类：
- 插件提供的 skill：迁移时要恢复对应插件来源、安装并启用插件
- 本地 `C:\Users\admin\.claude\skills` 里的 skill：迁移时要复制 `skills` 目录

### 一、插件提供的 skill
这些 skill 随插件恢复自动可用，命名形如 `plugin:skill-name`（例如 `superpowers:brainstorm`、`claude-hud:setup`、`commit-commands:commit`）。主要来源：`superpowers`（十余个规划/调试/TDD 相关 skill）、`claude-hud`、`commit-commands`、`code-review`。

关键点：
- **不要只记 skill 名称，要恢复它们背后的插件。** 先 marketplace，再安装，再启用。
- 当前启用插件对应的 skill 完整列表可在新机器运行 `claude plugin list` 查看，或翻对应 marketplace 仓库的 README。
- 若担心遗漏，在旧机器执行 `claude skill list > skills-backup.txt` 留底作参考。

### 二、本地 `skills` 目录提供的 skill
这些 skill 来自 `C:\Users\admin\.claude\skills`，迁移时复制整个 `skills` 目录即可恢复。

- `async-python-patterns`
- `ceeon-best-minds`
- `defuddle`
- `docx`
- `embedding-strategies`
- `fastapi-templates`
- `json-canvas`
- `obsidian-bases`
- `obsidian-cli`
- `obsidian-markdown`
- `pdf`
- `pdf-converter`
- `python-error-handling`
- `python-testing-patterns`
- `web-access` ⁎
- `webapp-testing`

注：上一节"其他插件提供的 skill"里也列了 `webapp-testing`，但本机 `skills\webapp-testing\` 目录同时存在，属于本地 skill；对应的插件 skill 已无依赖，直接迁本地目录即可。

⁎ `web-access` 是外部仓库 `eze-is/web-access`，不随本 repo 迁移。新机器单独执行：

```powershell
git clone https://github.com/eze-is/web-access C:\Users\admin\.claude\skills\web-access
```

运行时要求：启动浏览器（CentBrowser / Chrome 均可）时加 `--remote-debugging-port=9222` 参数。脚本按 `[9222, 9229, 9333]` fallback 探测，9222 开着即可连通。更新用 `cd skills\web-access; git pull`。

### 三、运行时目录（不建议迁）
- `C:\Users\admin\.claude\skills\learned` — 运行过程中积累的学习内容，属会话态数据，跨机意义不大。新机器会自己重新积累。

### 四、通常不需要单独考虑迁移的情况
- 如果某个 skill 来自插件，就不要试图"只迁 skill 不迁插件"。
- 如果某个 skill 来自本地 `skills` 目录，就不要一个个挑着复制，直接迁整个 `C:\Users\admin\.claude\skills` 更稳（`learned` 子目录可选择剔除）。

## 迁移前要准备什么
开始复制文件前，先准备这 5 项：

1. 无密钥版 `settings.json`
   - 按下文「敏感信息处理」的 `env` 三类划分处理：密钥和代理值改为占位符，通用开关保留实际值。

2. 已启用插件列表
   - 你平时真正用的是哪些插件，以当前启用状态为准。

3. marketplace 来源信息
   - 恢复插件前要先恢复来源。

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

### 按需迁移
- `C:\Users\admin\.claude\projects\*\memory`
- `settings.local.json`（如果实际存在）

说明：
- `memory` 只迁长期有价值的项目记忆。
- `plugins/` 目录本身不备份也不迁移（是 Claude CLI 运行时产物）；新机器按 `settings.json.enabledPlugins` + `extraKnownMarketplaces` 逐个重装即可。
- `settings.local.json` 定位是"机器本地覆盖层"，默认不跨机照搬；只有新机器路径结构接近时才考虑复用。
- **实际路径核对**：本机当前 `C:\Users\admin\.claude\` 下不存在该文件；如果存在，通常在 `C:\Users\admin\.claude\settings.local.json` 或项目目录内。早期版本文档提到的 `C:\Users\admin\.claude\.claude\settings.local.json` 已不再成立。

### 不要迁移
运行态、缓存、个人历史等短期数据：

- `C:\Users\admin\.claude\plugins\`（整个目录：Claude CLI 运行时产物，包含插件注册 / cache / marketplace gitlink / transcript 等，新机器 CLI 自动重建）
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
- `C:\Users\admin\.claude\rules_archive`、`skills_archive`（历史归档，非当前生效）
- 各项目目录下的 `tool-results`

## 敏感信息处理
迁移时，不要传输原始 `settings.json`。

### `env` 分三类处理

| 类别 | 示例键 | 处理方式 |
|------|--------|---------|
| **密钥/凭据** | `ANTHROPIC_AUTH_TOKEN`、`GITHUB_PERSONAL_ACCESS_TOKEN` | 必须在新机器重新填写，**不跨机复制值** |
| **本机代理/模型映射** | `ANTHROPIC_BASE_URL`（如 `http://127.0.0.1:8318`）、`ANTHROPIC_DEFAULT_HAIKU_MODEL`、`ANTHROPIC_DEFAULT_OPUS_MODEL`、`ANTHROPIC_DEFAULT_SONNET_MODEL` | 取决于新机器是否有同一代理服务；有则可保留，无则删掉 |
| **通用开关** | `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`、`CLAUDE_CODE_USE_POWERSHELL_TOOL` | 可直接跨机保留 |

### 正确做法
1. 先按三类拆分 `env`，只保留通用开关。
2. 记下密钥和代理的**键名**，不记录值。
3. 生成无密钥版 `settings.json`。
4. 新机器按键名清单重新填写。
5. 不通过网盘、压缩包、Git 仓库、聊天工具传输含密钥原文件。
6. 如果原文件或 token 曾被外发，迁移后立即轮换。

## 执行顺序
按下面顺序做，最稳。

### 第一步：从旧机器导出文件
导出：
- `CLAUDE.md`
- `rules`
- `skills`
- `agents`
- 无密钥版 `settings.json`
- `claude plugin list > plugins-snapshot.txt`（可选，用作新机核对参考）
- `projects\*\memory`（按需）

### 第二步：在新机器恢复基础文件
放置：
- `CLAUDE.md`
- `rules`
- `skills`
- `agents`
- 无密钥版 `settings.json`

### 第三步：恢复插件
注意顺序是三步，不要漏：
1. 先恢复 `extraKnownMarketplaces`
2. 再安装需要的插件
3. 最后根据 `enabledPlugins` 启用插件

### 第四步：重新填写环境变量
- 根据旧机器整理出来的 `env` 键名清单，在新机器重新填写。
- 不直接复制旧机器原始密钥文件。

### 第五步：最后再处理本机耦合项
这一步只做人工复核后再决定是否恢复，**每项都要核对新机器环境**：

- **`hooks`** — 当前本机示例调用 `C:\Windows\Media\Windows Notify.wav` 播放提示音。跨到 macOS / Linux 必须重写命令（不同 shell、不同音频工具、资源文件不存在）。
- **`statusLine`** — 当前本机依赖 `bash` + `awk` + `sort` + `tail` + `/c/Program Files/nodejs/node` + `claude-hud` 插件缓存路径。任一缺失或路径不同都要改。
- **`permissions.additionalDirectories`** — 当前本机含 `C:/Users/admin/.claude`、`C:\Users\admin\.claude\plans` 等绝对路径。换用户名、盘符或 WSL 路径都需手改。
- **`permissions.allow`** — 部分条目包含绝对路径（如 `Read(//c/Users/admin/Documents/...)`）或与本机 shell 绑定的 `PowerShell(...)` 规则；逐条判断是否仍成立。
- **`settings.local.json`** — 仅在文件实际存在且新机器路径结构接近时复用。

## 迁移后检查清单

### 基础文件
- `CLAUDE.md` 已到位
- `rules` 已到位
- `skills` 已到位
- `agents` 已到位
- `settings.json` 已写入无密钥版

### 插件
- marketplace 已恢复
- 需要的插件已重新安装
- 已启用插件和旧机器一致

### 环境变量
- 所需环境变量已在新机器重新填写
- 没有复用含密钥的旧配置文件

### 权限与路径
- `defaultShell` 正常
- `permissions` 中保留的规则仍有效
- 如启用了 `settings.local.json`，路径白名单仍成立

### 本机耦合项
- `hooks` 正常
- `statusLine` 正常
- Node / bash / PowerShell 可用

### 记忆
- 如果迁移了 `projects\*\memory`，对应项目可以读取长期 memory
- 没有迁移 `plans`、`sessions`、`tool-results` 等会话态目录

## 最后一句
这份执行版只回答"怎么迁"。如果你要判断"为什么不能整目录复制、哪些配置天生不适合跨机、哪些风险最容易踩"，看专家版。