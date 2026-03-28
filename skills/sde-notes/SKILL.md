---
name: sde-notes
description: 软考软件设计师备考笔记生成。当用户说「生成笔记」「第N章笔记」「X.X知识点笔记」「软考笔记」时触发。
---

# 软考笔记生成 Skill

## 脚本位置

```
C:\ObsidianNotes\ai应用开发\软件设计师支线\generate_notes.py
```

## 环境

- conda 环境：`notebooklm`（含 anthropic、python-dotenv，可选 notebooklm-py）
- 配置文件：`.env`（需填入 `ANTHROPIC_API_KEY`）

## 使用方式（只做单节，每节确认后再继续）

用户说「生成 X.X」时，在项目目录下运行：

```powershell
cd 'C:\ObsidianNotes\ai应用开发\软件设计师支线'
C:\Users\admin\.conda\envs\notebooklm\python.exe generate_notes.py --lesson X.X --overwrite
```

> 每次只生成一节。等用户在 Obsidian 中打开确认效果满意后，再继续下一节。

## 意图解析规则

| 用户说 | 执行参数 |
|--------|----------|
| 生成 1.9 的笔记 | `--lesson 1.9 --overwrite` |
| 生成 2.3 | `--lesson 2.3 --overwrite` |

## 执行流程

1. 从用户意图解析出节编号（如 `1.9`、`2.3`）
2. 检查 `.env` 中 `ANTHROPIC_API_KEY` 是否已填写
3. 运行脚本，实时显示输出
4. 告知用户笔记路径：`C:\ObsidianNotes\ai应用开发\软件设计师支线\笔记\`
5. 提示用户在 Obsidian 确认效果后告知是否继续

## 教材 Markdown 增强（当前启用）

教材 OCR 转 Markdown 文件已就绪，脚本会自动提取对应章节作为补充上下文：

```
C:\Download\SubBatch_2026-03-26-13-56-47_TXT\ai识别markdown_软件设计师教程第5版.md
```

无需额外配置，`generate_note()` 默认调用 `read_md_section()`，日志中会显示：
`[教材MD] 提取到 XXX 字内容`

## NotebookLM / PDF 路线（已废弃）

- PDF 为扫描版，`read_pdf_section()` 无法提取文字，已停用
- NotebookLM 模式需要浏览器登录，维护成本高，已停用
- 两者函数保留在代码中但 `generate_note()` 不再调用

## 字幕文件目录

`C:\Download\SubBatch_2026-03-26-13-56-47_TXT\`（84 个 TXT 文件，P1-P87）

有效课程文件格式：`P{N}【章.节 知识点名】`，如 `P11【1.8Cache】`。
考情分析（X.0）等特殊文件会被自动跳过。

## 输出位置

```
C:\ObsidianNotes\ai应用开发\软件设计师支线\笔记\
├── 第一章\
│   ├── 1.8 Cache.md
│   ├── 1.9 主存编址.md
│   └── 第一章-汇总.md
├── 第二章\
│   └── ...
```

## 首次使用检查

如果环境未配置，提示用户：

```powershell
conda create -n notebooklm python=3.12
conda activate notebooklm
pip install anthropic python-dotenv
```

然后编辑 `.env`，填入真实的 `ANTHROPIC_API_KEY`。
