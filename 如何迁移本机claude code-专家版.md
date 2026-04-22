# 如何迁移本机 Claude Code（专家版）

## 这份文档怎么用
这是一份决策版手册，回答的是：
- 为什么 Claude Code 不能简单整目录复制
- 哪些东西本质上适合迁移，哪些不适合
- 跨机时最常见的失效点在哪里
- 为什么执行版清单要按那个顺序做

配套文档：
- **当前环境快照**（本机配置分层全景）：`C:\Users\admin\.claude\README.md`
- **执行版**（照着执行的步骤清单）：`C:\Users\admin\Desktop\如何迁移本机claude code.md`

## 核心判断
Claude Code 的可迁移资产分 4 层：
1. 行为规则层
2. 能力定义层
3. 配置骨架层
4. 机器耦合层

前 3 层适合迁移，最后 1 层必须逐项复核，不能直接照搬。

## 为什么不能把 Claude Code 看成“一个配置文件”
如果把 Claude Code 理解成单个 `settings.json`，迁移时就很容易犯两个错误：
- 把真正重要的长期资产漏掉，例如 `CLAUDE.md`、`rules`、`skills`、`agents`
- 把本机耦合配置无脑复制过去，例如 `hooks`、`statusLine`、绝对路径白名单、含密钥的 `env`

迁移的关键不是“复制更多”，而是“区分哪些东西属于长期能力，哪些东西只属于这台机器”。

## 四层模型

### 1. 行为规则层
定义 Claude 如何工作、如何回答、如何遵守你的长期规范。

本机对应：
- `C:\Users\admin\.claude\CLAUDE.md`
- `C:\Users\admin\.claude\rules`

这一层最适合直接迁移，因为它表达的是你的长期协作方式，而不是某台机器的状态。

### 2. 能力定义层
定义 Claude 有哪些本地能力、任务方法、agent 行为。

本机对应：
- `C:\Users\admin\.claude\skills`
- `C:\Users\admin\.claude\agents`

这一层也适合直接迁移，因为它决定的是“你能做什么”。

### 3. 配置骨架层
定义插件启用状态、插件来源、默认 shell、部分权限策略。

本机对应：
- `C:\Users\admin\.claude\settings.json`（权威源：`enabledPlugins` + `extraKnownMarketplaces`）

这一层可以迁移，但前提是脱敏、拆分、筛选，只保留跨机仍成立的部分。

> `plugins/installed_plugins.json` 和 `known_marketplaces.json` 已证实是此设计层的冲突成品（跨机路径 / 时间戳 / commit hash 耦合），`plugins/` 目录整体不纳入 git，皆由 CLI 在新机重建。

### 4. 机器耦合层
定义那些依赖当前机器路径、命令、程序安装位置、白名单结构的配置。

本机典型内容：
- `settings.json` 里的 `hooks`
- `settings.json` 里的 `statusLine`
- `settings.json` 里的 `permissions.allow` 与 `permissions.additionalDirectories`
- `settings.local.json`（如果存在）

这一层不是"不能迁"，而是"不能默认迁"。任何跨机复用都应逐项验证。

> **路径核对注**：早期版本本文档将 `settings.local.json` 指为 `C:\Users\admin\.claude\.claude\settings.local.json`（双 `.claude` 嵌套），实际路径已不成立。若文件存在，通常在 `C:\Users\admin\.claude\settings.local.json` 或项目目录内。

## 本机当前插件状态说明
基于 `C:\Users\admin\.claude\settings.json.enabledPlugins` 字段（下表所列"默认状态"可通过旧机 `claude plugin list` 快照核对），状态需分三类而非两类。

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

三类区分的意义：
- **显式启用**才是你当前的实际工作集，属最小迁移目标。
- **显式禁用**反映过一个主动决定："装过但现阶段不用"，跨机价值低，可删。
- **默认状态**代表"装了但还没判断使不使"，跨机时不写入 `enabledPlugins` 即可保持状态。

> 早期版本本文档曾列出 `everything-claude-code@everything-claude-code`，当前已卸载，已不在任何分类内。

## 插件恢复为什么不是“勾上 enabledPlugins 就行”
插件恢复至少包含三段链路：
1. marketplace 来源正确
2. 插件实际安装成功
3. 插件启用状态恢复

少任何一步都不算真正恢复。

所以执行版才会要求按这个顺序执行：
- 先恢复 `extraKnownMarketplaces`
- 再安装插件
- 最后再按 `enabledPlugins` 启用

## 为什么整个 `env` 都应该视为敏感区
很多人迁移时只盯着 token，但实际风险来自整个 `env` 块。

原因有两个：
- 有些值本身就是凭据
- 有些值虽然不是密钥，但仍然暴露基础设施、代理地址、模型配置或内部接入方式

因此更稳妥的规则不是"删掉两个 token"，而是：
- 默认整个 `env` 都不跨机原样复制
- 只导出键名
- 在新机器重新填值

### env 三类实例划分
将当前 `env` 按跨机可保留度分三类，实际扭的是不同的风险权衡：

| 类别 | 示例键 | 为什么分这类 |
|------|--------|----------|
| **密钥/凭据** | `ANTHROPIC_AUTH_TOKEN`、`GITHUB_PERSONAL_ACCESS_TOKEN` | 同时是凭据和泄露成本最高的项；密钥轮换策略按此类执行 |
| **本机代理/模型映射** | `ANTHROPIC_BASE_URL`（例如 `http://127.0.0.1:8318`）、`ANTHROPIC_DEFAULT_*_MODEL` | 不是凭据但暴露接入拓扑；跨机能否复用取决于新机器是否运行相同代理 |
| **通用开关** | `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`、`CLAUDE_CODE_USE_POWERSHELL_TOOL` | 纯行为开关，无敏感信息，可直接跨机 |

这里的关键判断不是"全部删掉"而是"按风险类别重新赋值"：密钥类不复制值、代理类核实后保留或修改、开关类直接继承。

## 绝对路径为什么是跨机头号风险
你当前环境里已经出现多种绝对路径耦合，每一类都有明确位置：

| 耦合类型 | 当前本机实例 | 失效触发条件 |
|---------|-------------|-------------|
| `permissions.additionalDirectories` | `C:/Users/admin/.claude`、`C:\Users\admin\.claude\plans` | 用户名 / 盘符 / WSL 路径格式变化 |
| `permissions.allow` | `Read(//c/Users/admin/Documents/...)`、`PowerShell(Get-ChildItem:*)` | 路径不存在或 shell 不为 PowerShell |
| `hooks` 资源文件 | `C:\Windows\Media\Windows Notify.wav` | 非 Windows 系统该文件不存在 |
| `statusLine` 外部命令 | `/c/Program Files/nodejs/node`、`bash`、`awk`、`sort`、`tail` | Node 安装位置变化或 GNU 工具链不存在 |
| `statusLine` 缓存路径 | `${CLAUDE_CONFIG_DIR}/plugins/cache/claude-hud/claude-hud/*` | `claude-hud` 未安装或版本不同 |

这些不是"边角问题"，而是跨机迁移最常见故障源。

## 为什么 `hooks`、`statusLine`、`settings.local.json` 不该直接照搬
它们的问题不在于"语法复杂"，而在于它们天然绑定机器环境。

### `hooks`
当前本机的 `Stop` hook：
```
(New-Object Media.SoundPlayer 'C:\Windows\Media\Windows Notify.wav').PlaySync()
```
同时绑定三件事：PowerShell 作为 shell、`.NET` `Media.SoundPlayer` 类、Windows 系统自带音频文件。其中任一条件换到新平台都失效。

### `statusLine`
当前本机调用链：`bash` → `ls` 遍历 → `awk` 拆分 → `sort` 按版本 → `tail` 取最新 → `/c/Program Files/nodejs/node` 跑 `claude-hud` 插件的 `dist/index.js`。上链除了外部工具，还绑定特定版本的插件缓存目录。

### `settings.local.json`
它的定位本来就更接近"机器本地覆盖层"，不是跨机通用配置层。

所以这三类内容最合理的处理方式不是"默认迁移"，而是"最后一层再逐项判断"。

## 迁移时应该保留什么，不保留什么

### 应该优先保留
- `CLAUDE.md`
- `rules`
- `skills`
- `agents`
- 无密钥版 `settings.json`
- 有长期价值的 `projects\*\memory`

### 不该优先保留
- 插件缓存
- sessions
- history
- plans
- tool-results
- debug / metrics / browser-profiles / shell-snapshots

原因很简单：
前一组定义长期工作方式与能力，后一组只是运行痕迹、缓存或会话态数据。

## 迁移失败最常见的 5 个原因

### 1. 复制了原始 `settings.json`
结果把敏感 `env` 一并带走，形成泄露风险。

### 2. 只恢复了启用标记，没有恢复插件来源与安装
表面上配置没问题，实际插件并不存在。

### 3. 旧机器绝对路径被原样复制
新机器用户名、盘符或安装路径一变，配置立刻失效。

### 4. 低估了外部依赖
没有 bash、Node、PowerShell 或 GNU 工具链时，部分能力会直接坏掉。

### 5. 混淆了长期资产和会话态数据
把 `plans`、`sessions`、`tool-results` 一并带走，不但没收益，还会引入脏状态。

## 为什么执行版是那个顺序
执行版的顺序不是随便排的，而是为了降低失败率：

1. 先恢复行为规则层和能力定义层
   - 因为这两层稳定、低风险、长期有效

2. 再恢复配置骨架层
   - 因为插件来源、插件安装、启用状态必须按依赖顺序恢复

3. 再重新填写 `env`
   - 因为密钥应在新机器注入，而不是从旧机器复制原值

4. 最后再处理机器耦合层
   - 因为这部分最依赖新机器的真实环境，必须在前面稳定后再判断

## 你该怎么选迁移策略

### 如果你的目标是"最快恢复可用"
照执行版清单走：
- 只迁最关键层
- 先跑起来
- 再逐项补高保真内容

### 如果你的目标是“尽量高保真复刻旧环境”
也不要整目录复制。
仍然应该按四层模型来，只是：
- 第 4 层复核得更细
- 会迁更多按需内容
- 会多做环境兼容检查

## 最终结论
Claude Code 的迁移不是“复制 `.claude` 目录”，而是“按层拆分资产”：
- 行为规则层直接迁
- 能力定义层直接迁
- 配置骨架层脱敏后迁
- 机器耦合层最后复核

执行版负责回答"怎么迁"；这份专家版负责回答"为什么这样迁"。