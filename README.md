# Claude Code 全局配置总览

> 这份文档记录当前 `C:\Users\admin\.claude` 下**已核实**的全局配置结构、文档分工和迁移边界。
> 它不是永久正确的环境快照；涉及插件启用、安装和本地覆盖时，应以配置文件现状为准。

---

## 1. 配置分层

当前全局 Claude Code 配置可以按下面几层理解：

1. `CLAUDE.md`
   - 定义长期协作方式、输出风格和执行哲学。
   - 这是行为约束层，不是运行时状态快照。

2. `rules/`
   - 定义通用工程规范，如编码、测试、git、security、agents。
   - 这里描述的是规则，不是插件或技能安装状态。

3. `settings.json`
   - 当前全局主配置入口。
   - 已核实包含这些配置类别：`env`、`permissions`、`hooks`、`defaultShell`、`statusLine`、`enabledPlugins`、`extraKnownMarketplaces`、`theme` 等。

4. `.claude/settings.local.json`
   - 当前存在于 `C:\Users\admin\.claude\.claude\settings.local.json`。
   - 这是本机本地覆盖层，适合放机器相关或更私有的补充配置；不能默认当作跨机通用配置。

5. `plugins/installed_plugins.json`
   - 记录的是**已安装插件**，不是“当前启用插件”的唯一来源。
   - 文档写插件状态时，必须和 `enabledPlugins` 区分开。

6. `skills/` 与插件内 `skills/`
   - 本地 skills 来自 `C:\Users\admin\.claude\skills\`。
   - 插件提供的 skills 来自已启用插件的缓存目录。
   - 两类来源都可能出现在技能列表中，不能混写成同一来源。

## 2. 当前已核实的关键配置入口

### 主配置文件
- `C:\Users\admin\.claude\settings.json`

### 本地覆盖文件
- `C:\Users\admin\.claude\.claude\settings.local.json`

### 插件安装登记
- `C:\Users\admin\.claude\plugins\installed_plugins.json`

### 规则目录
- `C:\Users\admin\.claude\rules\`

### 本地 skills 目录
- `C:\Users\admin\.claude\skills\`

## 3. 插件状态怎么判断

插件状态要分三层看：

### 3.1 已安装插件
权威来源：`plugins/installed_plugins.json`

当前已核实包含这些插件记录：
- `frontend-design@claude-plugins-official`
- `context7@claude-plugins-official`
- `feature-dev@claude-plugins-official`
- `kotlin-lsp@claude-plugins-official`
- `codex@openai-codex`
- `playwright@claude-plugins-official`
- `pyright-lsp@claude-plugins-official`
- `claude-hud@claude-hud`
- `superpowers@claude-plugins-official`
- `github@claude-plugins-official`
- `skill-creator@claude-plugins-official`
- `claude-md-management@claude-plugins-official`（`local scope`，绑定特定项目路径）

### 3.2 显式启用/禁用插件
权威来源：`settings.json.enabledPlugins`

当前显式启用：
- `context7@claude-plugins-official`
- `pyright-lsp@claude-plugins-official`
- `claude-hud@claude-hud`
- `superpowers@claude-plugins-official`
- `github@claude-plugins-official`
- `skill-creator@claude-plugins-official`

当前显式禁用：
- `codex@openai-codex`
- `feature-dev@claude-plugins-official`
- `frontend-design@claude-plugins-official`

### 3.3 已安装但未显式声明
这类插件出现在 `installed_plugins.json`，但不在 `enabledPlugins` 中。

当前至少包括：
- `kotlin-lsp@claude-plugins-official`
- `playwright@claude-plugins-official`
- `claude-md-management@claude-plugins-official`

说明：
- “已安装”不等于“显式启用”。
- “未显式声明”不应在文档里被写成“当前启用”。
- `claude-md-management` 是项目级本地插件，不应写成全局通用插件基线。

## 4. 当前已核实的 marketplace 来源

权威来源：`settings.json.extraKnownMarketplaces`

当前只核实到这 3 个来源：

| 市场 ID | 来源 |
|---------|------|
| `claude-hud` | GitHub `jarrodwatts/claude-hud` |
| `claude-plugins-official` | git `https://github.com/anthropics/claude-plugins-official.git` |
| `openai-codex` | GitHub `openai/codex-plugin-cc` |

除非配置文件再次变化，否则不应继续把其他 marketplace 写成“当前全局已配置来源”。

## 5. 本地 skills 与插件 skills 的边界

### 本地 skills
当前仓库中可见的本地 skill 目录包括：
- `async-python-patterns`
- `ceeon-best-minds`
- `defuddle`
- `pdf`
- `pdf-converter`
- `python-error-handling`
- `python-testing-patterns`
- `web-access`

### 插件侧能力来源
插件除了启用 MCP、命令或其他运行时能力，也可能提供 skills；但具体暴露什么能力，应以对应插件当前内容为准，不应只凭插件名在文档里静态推断。

文档里提到某个能力时，最好同时写明它来自“本地 skills 目录”还是“插件侧提供”，避免迁移时只记名字不记来源。

## 6. 当前文档分工

- `README.md`
  - 负责说明当前全局配置结构、术语和文档导航。
- `如何迁移本机claude code.md`
  - 执行版迁移清单，回答“按什么顺序迁”。
- `如何迁移本机claude code-专家版.md`
  - 决策版迁移说明，回答“为什么这么迁、哪些东西不该直接搬”。
- `rules/README.md`
  - 只说明 `rules/` 目录本身，不再承担全局插件/skills 快照职责。

## 7. 迁移与维护边界

### 适合长期维护进文档的内容
- 配置入口路径
- 配置层级关系
- 字段职责
- 插件状态的判断方法
- 迁移边界和风险点

### 不适合写成静态事实的内容
- token、密钥、完整 env 值
- 机器专属路径白名单细节
- 项目专属插件被误写成全局标准配置
- 已经过时的 skills/plugins/rules 快照

## 8. 使用这份目录时的判断准则

遇到“当前到底启用了什么、从哪里来的、该不该迁移”这类问题时，按这个顺序核查最稳：

1. 先看 `settings.json`
2. 再看 `.claude/settings.local.json`
3. 再看 `plugins/installed_plugins.json`
4. 最后再看文档是否需要同步更新

这样可以避免把旧文档快照误当成当前真实状态。
