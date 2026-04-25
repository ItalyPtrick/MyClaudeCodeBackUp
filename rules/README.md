# Rules

## 作用
`rules/` 用来存放 Claude 在不同项目里应长期遵守的工程规范。

这些规则回答的是“应该怎么做”，不是“当前插件装了什么”或“环境里有哪些 skill”。

## 当前目录结构

当前仓库内已核实的规则目录如下：

```
rules/
├── README.md
└── common/
    ├── agents.md
    ├── coding-style.md
    ├── git-workflow.md
    ├── security.md
    └── testing.md
```

## `common/` 的职责
`common/` 存放所有项目都适用的通用规范：

- `coding-style.md`：编码风格与实现约束
- `git-workflow.md`：commit / PR 工作流
- `testing.md`：测试要求与 TDD 约束
- `security.md`：安全检查与密钥管理
- `agents.md`：子代理使用原则

## 当前边界
这份 README 只描述当前 `rules/` 目录本身，不再承担以下职责：

- 维护插件清单
- 维护 skills 快照
- 维护多语言规则目录的历史设想
- 充当全局环境总览

这些内容应分别查看：
- 全局配置总览：`C:\Users\admin\.claude\README.md`
- 执行版迁移说明：`C:\Users\admin\.claude\如何迁移本机claude code.md`
- 专家版迁移说明：`C:\Users\admin\.claude\如何迁移本机claude code-专家版.md`

## 如何维护
新增或调整规则时，优先遵守这几个原则：

1. 规则文件只写长期有效的工程约束。
2. 不把临时环境状态、插件快照、迁移步骤混进 `rules/`。
3. 如果规则只对单一语言或单一项目成立，应单独建目录或单独建文档，不要冒充通用规则。
4. 如果 README 与实际目录结构不一致，先修 README，再继续扩展。

## 未来扩展
如果后续真的增加语言专属规则目录，应在目录实际落地后再把它写进 README，而不是提前把设想写成现状。
