# 全局规范（学习阶段版）

## 环境

- OS: Windows 11, PowerShell 终端
- Python: 3.12.10, Miniconda 包管理
- 编辑器: VS Code + Black 格式化
- 路径分隔符：优先正斜杠 /

## 当前阶段目标

**把项目跑起来，代码能跑、能展示、能上 GitHub。**
不追求完美架构，不追求测试覆盖率，先做出来再优化。

---

## 会话启动协议

每次新会话开始，按顺序执行：

1. 读取本文件（CLAUDE.md）
2. 读取 `progress.txt` 了解当前进度
3. 确认："已同步进度：[当前状态摘要]，准备开始。"

> 如果 progress.txt 不存在，提示我创建。

---

## 代码规范

### 基本要求

- Python 3.12，类型注解尽量写但不强制
- 函数职责单一，太长就拆，没有硬性行数限制
- 用 Black 格式化，保持代码整洁
- 变量和函数名用英文，注释可以中文

### 依赖管理

- 用 conda 管理虚拟环境，每个项目独立环境
- 依赖记录在 requirements.txt
- API key 存 .env 文件，.env 加入 .gitignore，绝不提交到 Git

### 报错处理

- 遇到报错先读错误信息，不要直接绕过
- 可以用宽泛的 except，但要打印错误内容方便调试：
  ```python
  except Exception as e:
      print(f"出错了: {e}")
  ```

---

## 技术栈

```
当前学习中：
  Python 基础语法

即将用到：
  FastAPI          → 写后端 API
  Streamlit        → 做网页 Demo
  大模型 API        → OpenAI / 国内模型直接调用
  LangChain        → 先学会直接调 API，再考虑用它
  ChromaDB         → 向量数据库，做 RAG 时用

暂时不需要：
  Numpy / Pandas / Matplotlib（数据科学方向，不是你的目标）
  Django / Flask（FastAPI 够用）
  单元测试（项目跑通后再补）
```

---

## Git 规范（够用就行）

```powershell
git init
git add .
git commit -m "feat: 描述做了什么"
git push
```

提交信息前缀：
- `feat:` 新功能
- `fix:` 修 bug
- `docs:` 改文档

**每完成一个功能就 commit 一次，别攒着。**

---

## progress.txt 格式

每次结束前更新，记录在哪、下次从哪继续：

```
== 最后更新 ==
[日期]

== 今天完成了 ==
- 

== 下次继续 ==
- 

== 卡住的问题 ==
- 
```

---

## 常用命令

```powershell
# 环境
conda create -n [项目名] python=3.12
conda activate [项目名]

# 安装依赖
pip install -r requirements.txt
pip freeze > requirements.txt

# 启动
uvicorn main:app --reload     # FastAPI
streamlit run app.py          # Streamlit

# 格式化
black .
```

---

## 安全底线（只有两条）

1. `.env` 文件里放 API key，永远不提交到 Git
2. 新建项目第一步就创建 `.gitignore`，写上 `.env`

---

## 禁止事项（精简版）

- 不把 API key 写进代码
- 不直接推送到 main 分支
- 不在不认识的目录下启动 Claude Code

---

## 自我改进

我纠正你后，把教训追加到 `lessons.md`：

```
### [日期] [错误简述]
错误：[什么情况触发]
正确：[应该怎么做]
```

---

## 升级提示

**当你完成两个完整项目、准备投简历时，切换到 CLAUDE_pro.md。**
那个版本包含：TDD、三层架构、代码审查、PR 规范等完整工程规范。
