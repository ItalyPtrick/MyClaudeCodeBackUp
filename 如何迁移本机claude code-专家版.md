# 如何迁移本机 Claude Code（专家版）

## 这份文档怎么用
这是一份决策版手册，回答的是：
- 为什么 Claude Code 不能简单整目录复制
- 哪些东西本质上适合迁移，哪些不适合
- 跨机时最常见的失效点在哪里
- 为什么执行版清单要按那个顺序做

配套文档：
- **当前全局配置总览**：`C:\Users\admin\.claude\README.md`
- **执行版**：`C:\Users\admin\.claude\如何迁移本机claude code.md`

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

这一层最适合直接迁移，因为它表达的是长期协作方式，而不是某台机器的状态。

### 2. 能力定义层
定义 Claude 有哪些本地能力、任务方法、agent 行为。

本机对应：
- `C:\Users\admin\.claude\skills`
- `C:\Users\admin\.claude\agents`

这一层也适合直接迁移，因为它决定的是“你能做什么”。

### 3. 配置骨架层
定义插件来源、显式启用/禁用状态、默认 shell、通用权限框架等。

本机对应：
- `C:\Users\admin\.claude\settings.json`
- `C:\Users\admin\.claude\plugins\installed_plugins.json`

但这两者承担的职责不同：
- `settings.json` 用来表达配置骨架，尤其是 `enabledPlugins`、`extraKnownMarketplaces`、`defaultShell` 等。
- `installed_plugins.json` 只是“已安装插件登记”，不应被误当成“当前启用配置”。

这一层可以迁移，但前提是脱敏、拆分、筛选，只保留跨机仍成立的部分。

### 4. 机器耦合层
定义那些依赖当前机器路径、命令、程序安装位置、白名单结构的配置。

本机典型内容：
- `settings.json` 里的 `hooks`
- `settings.json` 里的 `statusLine`
- `settings.json` 里的 `permissions.allow` 与 `permissions.additionalDirectories`
- `C:\Users\admin\.claude\.claude\settings.local.json`

这一层不是“不能迁”，而是“不能默认迁”。任何跨机复用都应逐项验证。

## 为什么要把 `settings.json`、`settings.local.json`、`installed_plugins.json` 分开看
这三类文件最容易在文档里被混写。

### `settings.json`
它是当前全局主配置入口，负责表达：
- 主行为开关
- marketplace 来源
- 显式启用/禁用插件
- hooks / statusLine / shell 等运行配置

### `.claude/settings.local.json`
当前本机该文件存在于：
- `C:\Users\admin\.claude\.claude\settings.local.json`

它属于本地覆盖层，适合放更私有或更机器相关的补充配置。跨机时，它默认不是“应该直接复制”的对象。

### `installed_plugins.json`
它回答的是“哪些插件装过”，不是“当前以什么状态工作”。

所以文档里提插件状态时，至少要区分这 3 类事实：
1. 已安装
2. 显式启用
3. 显式禁用

否则迁移时最容易出现“看起来配了，其实没装”或“装了，但文档误写成已启用”。

## 本机当前插件状态说明
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

三类区分的意义：
- **显式启用**才是当前主工作集。
- **显式禁用**说明曾经装过，但当前不纳入主工作流。
- **已安装但未显式声明**表示装过，但文档不应把它写成“当前启用”。

额外注意：
- `claude-md-management` 当前是 `local scope` 插件，绑定特定 `projectPath`，不能写成所有项目共享的全局插件标准。

## 当前 marketplace 来源为什么也要单独核实
插件恢复至少包含三段链路：
1. marketplace 来源正确
2. 插件实际安装成功
3. 插件启用状态恢复

少任何一步都不算真正恢复。

当前 `settings.json.extraKnownMarketplaces` 已核实只有这 3 个：
- `claude-plugins-official`
- `claude-hud`
- `openai-codex`

所以迁移时不能继续把其他 marketplace 当作“当前机器已配置来源”。如果新机器需要额外来源，应作为新增决策，而不是按旧快照无脑复制。

## 为什么整个 `env` 都应该视为敏感区
很多人迁移时只盯着 token，但实际风险来自整个 `env` 块。

原因有两个：
- 有些值本身就是凭据
- 有些值虽然不是密钥，但仍然暴露基础设施、代理地址、模型映射或内部接入方式

因此更稳妥的规则不是“删掉两个 token”，而是：
- 默认整个 `env` 都不跨机原样复制
- 只导出键名
- 在新机器重新填值

### env 三类实例划分
| 类别 | 示例键 | 为什么分这类 |
|------|--------|-------------|
| **密钥/凭据** | `ANTHROPIC_AUTH_TOKEN`、`GITHUB_PERSONAL_ACCESS_TOKEN` | 同时是凭据和泄露成本最高的项 |
| **本机代理/模型映射** | `ANTHROPIC_BASE_URL`、`ANTHROPIC_DEFAULT_*_MODEL` | 不是凭据，但暴露接入拓扑；能否复用取决于新机器环境 |
| **通用开关** | `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`、`CLAUDE_CODE_USE_POWERSHELL_TOOL` | 行为开关；目标机仍需要同样行为且环境兼容时可继承 |

## 绝对路径为什么是跨机头号风险
当前环境里已经出现多种绝对路径耦合，每一类都有明确位置：

| 耦合类型 | 典型来源 | 失效触发条件 |
|---------|---------|-------------|
| `permissions.additionalDirectories` | 主配置或本地覆盖中的绝对路径 | 用户名、盘符、路径结构变化 |
| `permissions.allow` | 含具体路径或 shell 前提的白名单 | 路径不存在或 shell 环境变化 |
| `hooks` 资源文件 | 本机音频、脚本或命令路径 | 换平台后资源不存在 |
| `statusLine` 外部命令 | `bash`、Node、插件缓存路径 | 工具链或缓存目录变化 |
| 本地覆盖文件 | `.claude/settings.local.json` | 新机器环境与旧机器不一致 |

这些不是边角问题，而是跨机迁移最常见故障源。

## 为什么 `hooks`、`statusLine`、`settings.local.json` 不该直接照搬
它们的问题不在于“语法复杂”，而在于它们天然绑定机器环境。

### `hooks`
当前本机 `settings.json` 中包含 `Stop` hook。只要换平台、换 shell 或换资源路径，就可能失效。

当前本机 hook 依赖 PowerShell + .NET `Media.SoundPlayer` + `C:\Windows\Media\Windows Notify.wav`，三者缺一即失效。

### `statusLine`
当前本机 `statusLine` 通过外部命令拼接 `claude-hud` 的缓存目录和运行时路径，天然依赖：
- `bash`
- Node.js
- 插件缓存目录结构
- 当前机器上的可执行路径

当前本机调用链依赖 bash、Node.js（`/c/Program Files/nodejs/node`）、awk/sort/tail 以及 claude-hud 插件缓存目录，任一条件变化即失效。

### `settings.local.json`
它本来就更接近“本机覆盖层”，不应被当成跨机通用模板。

所以这三类内容最合理的处理方式不是“默认迁移”，而是“最后一层再逐项判断”。

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

### 2. 只恢复了启用标记，没有恢复 marketplace 和安装
表面上配置没问题，实际插件并不存在。

### 3. 旧机器绝对路径被原样复制
新机器用户名、盘符或安装路径一变，配置立刻失效。

### 4. 低估了外部依赖
没有 bash、Node、PowerShell 或其他依赖时，部分能力会直接坏掉。

### 5. 混淆了长期资产和会话态数据
把 `plans`、`sessions`、`tool-results` 一并带走，不但没收益，还会引入脏状态。

## 为什么执行版是那个顺序
执行版的顺序不是随便排的，而是为了降低失败率：

1. 先恢复行为规则层和能力定义层
   - 因为这两层稳定、低风险、长期有效

2. 再恢复配置骨架层
   - 因为 marketplace、插件安装、启用状态有依赖顺序

3. 再重新填写 `env`
   - 因为密钥应在新机器注入，而不是从旧机器复制原值

4. 最后再处理机器耦合层
   - 因为这部分最依赖新机器真实环境，必须在前面稳定后再判断

## 你该怎么选迁移策略
### 如果目标是“最快恢复可用”
照执行版清单走：
- 只迁最关键层
- 先跑起来
- 再逐项补高保真内容

### 如果目标是“尽量高保真复刻旧环境”
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

执行版负责回答“怎么迁”；这份专家版负责回答“为什么这样迁”。
