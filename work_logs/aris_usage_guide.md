# ARIS 工作流使用指南

> 让 Claude Code 在你睡觉时做科研。醒来发现论文已被打分、弱点已被定位、实验已跑完、叙事已重写——全自动。

---

## 目录

- [启动 ARIS](#启动-aris)
- [基础命令](#基础命令)
- [四大工作流](#四大工作流)
- [常用参数](#常用参数)
- [当前配置](#当前配置)
- [故障排除](#故障排除)

---

## 启动 ARIS

### 方式一：直接启动（推荐在项目目录下）

```bash
# 进入研究项目目录
cd /data1/user/fangjiukai/Pointcept-main  # 或其他研究项目

# 启动 ARIS（默认使用 workspace-write 权限）
/data1/user/fangjiukai/.local/bin/aris
```

### 方式二：使用 tmux（长期运行）

```bash
# 创建新 tmux 会话
tmux new -s aris

# 在 tmux 中启动 ARIS
cd /data1/user/fangjiukai/Pointcept-main
/data1/user/fangjiukai/.local/bin/aris

# 后续重新连接
tmux attach -t aris
```

### 方式三：一行命令启动

```bash
tmux new-session -d -s aris "cd /data1/user/fangjiukai/Pointcept-main && /data1/user/fangjiukai/.local/bin/aris"
tmux attach -t aris
```

---

## 基础命令

| 命令 | 功能 | 示例 |
|------|------|------|
| `/help` | 显示所有可用 slash 命令 | `/help` |
| `/status` | 查看当前会话状态（模型、权限等） | `/status` |
| `/compact` | 压缩本地会话历史，释放上下文 | `/compact` |
| `/model gpt-5.4` | 选择执行模型 | `/model deepseek-v4-pro` |
| `/reviewer deepseek-v4-pro` | 选择审稿模型 | `/reviewer gpt-5.4` |
| `/setup` | 重新配置 API keys 和模型 | `/setup` |
| `/plan` | 进入只读计划模式 | `/plan execute` 或 `/plan exit` |
| `/tasks` | 查看或管理持久任务列表 | `/tasks` |
| `/skills list` | 列出所有可用 skills | `/skills list` |
| `/skills show <name>` | 显示特定 skill 详情 | `/skills show auto-review-loop` |
| `/skills export <name>` | 导出 skill 到文件 | `/skills export idea-discovery` |
| `/permissions` | 查看/切换权限模式 | `/permissions workspace-write` |
| `/mcp list` | 列出已加载的 MCP servers | `/mcp list` |

---

## 四大工作流

### 工作流 1：Idea 发现与方案精炼

> "这个领域最新进展是什么？哪里有 gap？怎么解决？"

**一键调用：**
```
/idea-discovery "研究方向"
```

**分步调用：**
```bash
# 1. 文献调研（多源搜索）
/research-lit "离散扩散语言模型"
/research-lit "topic" — sources: deepxiv, web        # 指定文献源
/research-lit "topic" — arxiv download: true          # 同时下载 PDF

# 2. 头脑风暴 idea
/idea-creator "研究方向"

# 3. 查新验证
/novelty-check "top idea"

# 4. 深度评审
/research-review "top idea"

# 5. 精炼方案
/research-refine "top idea"

# 6. 实验规划
/experiment-plan
```

**输出：**
- `IDEA_REPORT.md` — 排名后的 idea 列表
- `refine-logs/FINAL_PROPOSAL.md` — 精炼后的方案
- `refine-logs/EXPERIMENT_PLAN.md` — 实验路线图

---

### 工作流 1.5：实验桥接

> "我有计划了，帮我实现代码、部署实验、拿到初始结果。"

**一键调用：**
```
/experiment-bridge
/experiment-bridge "my_plan.md"    # 指定计划文件
```

**分步调用：**
```bash
# 1. 解析实验计划
/experiment-bridge

# 2. 部署到 GPU
/run-experiment

# 3. 监控进度
/monitor-experiment
```

**涉及 Skills：** `experiment-bridge` + `run-experiment` + `monitor-experiment`

---

### 工作流 2：自动 review 循环（睡觉时跑）

> "帮我 review 论文，修复问题，循环到通过为止。"

**一键调用：**
```
/auto-review-loop "论文主题"
/auto-review-loop "重点看第 3-5 节，CRF 结果偏弱"  # 指定范围
```

**可选参数：**
```bash
/auto-review-loop "主题" — difficulty: nightmare    # 极限审稿强度
/auto-review-loop "主题" — human checkpoint: true   # 每轮暂停等你确认
/auto-review-loop "主题" — effort: max              # 最大工作强度
```

**工作原理：**
```
外部 LLM 评审 → Claude Code 实现修复 → /run-experiment 部署 → 收结果 → 再评审 → 循环
```

**审稿难度等级：**
| 难度 | 变化 | 适用场景 |
|------|------|---------|
| `medium`（默认） | 标准 MCP review | 日常使用 |
| `hard` | + Reviewer Memory + 辩论协议 | 想要更严格的反馈 |
| `nightmare` | + GPT 通过 `codex exec` 直读代码 | 投顶会前的极限压测 |

---

### 工作流 3：论文写作

> "把我的研究报告变成可投稿的 PDF。"

**输入：** `NARRATIVE_REPORT.md` — 研究叙事文档

**一键调用：**
```
/paper-writing "NARRATIVE_REPORT.md"
```

**分步调用：**
```bash
# 1. 生成大纲+claims矩阵
/paper-plan

# 2. 生成图表
/paper-figure

# 3. 逐节写 LaTeX
/paper-write

# 4. 编译 PDF
/paper-compile

# 5. 自动润色（2轮审稿+格式检查）
/auto-paper-improvement-loop
```

**输出：** `paper/` 目录，含 LaTeX 源码、干净的 `.bib`、编译好的 PDF

---

### 工作流 4：Rebuttal

> "审稿意见来了？ARIS 帮你起草安全的 rebuttal。"

```bash
/rebuttal "paper/ + reviews" — venue: ICML, character limit: 5000
```

**参数：**
| 参数 | 默认值 | 说明 |
|------|--------|------|
| `venue` | `ICML` | 目标会议 |
| `character limit` | — | **必填** 字符限制 |
| `quick mode` | `false` | 仅解析 + 策略（Phase 0-3） |
| `auto experiment` | `false` | 自动跑补充实验 |
| `max stress test rounds` | `1` | GPT-5.4 压力测试轮数 |
| `max followup rounds` | `3` | 每个 reviewer follow-up 上限 |

**三道安全门：**
- 🔒 **不编造** — 每句话有出处
- 🔒 **不过度承诺** — 没批准的不承诺
- 🔒 **全覆盖** — 每个审稿意见都追踪

**输出：** `PASTE_READY.txt`（精确字数）+ `REBUTTAL_DRAFT_rich.md`（详细版）

---

### 全流程：端到端科研

```bash
/research-pipeline "研究方向"                    # 从 idea 到投稿
/research-pipeline "研究方向" — effort: beast    # 最大工作强度
```

---

## 常用参数

### 全局参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `AUTO_PROCEED` | `true` | 自动继续（设为 `false` 手动审批） |
| `human checkpoint` | `false` | 每轮后暂停等你确认 |
| `effort` | `balanced` | 工作强度：`lite`(0.4x) / `balanced`(1x) / `max`(2.5x) / `beast`(5-8x) |
| `sources` | `all` | 文献源：`zotero` / `obsidian` / `local` / `web` / `semantic-scholar` / `deepxiv` / `exa` |
| `arxiv download` | `false` | 下载最相关的 arXiv PDF |
| `difficulty` | `medium` | 审稿强度：`medium` / `hard` / `nightmare` |
| `code review` | `true` | GPT-5.4 代码审查（设为 `false` 跳过） |
| `DBLP_BIBTEX` | `true` | 从 DBLP/CrossRef 获取真实 BibTeX |
| `compact` | `false` | 生成精简摘要文件 |

### 示例

```bash
# 在 idea 选择关卡暂停
/research-pipeline "方向" — AUTO_PROCEED: false

# 每轮 review 后暂停
/research-pipeline "方向" — human checkpoint: true

# 只搜特定文献源
/research-pipeline "方向" — sources: zotero, web

# 极限审稿强度
/auto-review-loop "论文" — difficulty: nightmare

# 组合使用
/research-pipeline "方向" — AUTO_PROCEED: false, human checkpoint: true, effort: max
```

---

## 当前配置

### 模型配置

| 组件 | 模型 | API 地址 |
|------|------|----------|
| **Executor** | `gpt-5.4` | `https://w.ciykj.cn/v1` |
| **Reviewer** | `deepseek-v4-pro` | `https://api.deepseek.com` |
| **Codex CLI** | `gpt-5.5` | `https://w.ciykj.cn` |

### 权限配置

| 配置 | 值 |
|------|-----|
| **默认权限** | `workspace-write` |
| **MCP Server** | `llm-chat` (DeepSeek) |

### 关键文件位置

```
~/.codex/config.toml          # Codex CLI 配置
~/.config/aris/config.json    # ARIS executor/reviewer 配置
~/.config/aris/aris.env        # ARIS executor 环境变量
~/.local/bin/aris             # ARIS wrapper 启动脚本
~/.codex/mcp-servers/llm-chat/ # DeepSeek MCP server
/data1/user/fangjiukai/Auto-claude-code-research-in-sleep/  # ARIS 工作流源码
```

---

## 故障排除

### ARIS 启动后显示 setup 菜单

```bash
# 清理 shell 命令缓存
hash -r

# 使用绝对路径启动
/data1/user/fangjiukai/.local/bin/aris

# 或选择 OpenAI provider，base URL 用 https://api.deepseek.com/v1
```

### API 连通性测试

```bash
# 测试 w.ciykj.cn (GPT)
curl -s https://w.ciykj.cn/v1/models -H "Authorization: Bearer sk-5cab65e4a..."

# 测试 DeepSeek
curl -s https://api.deepseek.com/v1/models -H "Authorization: Bearer sk-a94735c7..."

# 测试 llm-chat MCP
cat /tmp/llm-chat-mcp-debug.log | tail -20
```

### 清理残留进程

```bash
# 清理残留的 MCP server 进程
pkill -f "llm-chat/server.py"

# 清理残留的 ARIS 进程
pkill -f "aris"
```

### 检查状态

```bash
# 在 ARIS 中检查
/status
/mcp list
/models
```

---

## 快捷命令参考

```bash
# 启动
aris                                            # 默认项目目录
cd /data1/user/fangjiukai/vision && aris          # 指定项目
tmux new -s aris && aris                          # 后台运行

# 文献调研
/research-lit "topic"
/research-lit "topic" — sources: deepxiv

# Idea 发现
/idea-discovery "研究方向"
/idea-creator "方向"
/research-refine "idea"

# 实验
/experiment-bridge
/run-experiment
/monitor-experiment

# Review
/auto-review-loop "论文主题"
/research-review "主题"

# 论文写作
/paper-writing "NARRATIVE_REPORT.md"
/paper-plan
/paper-write

# Rebuttal
/rebuttal "paper/" — venue: ICML, character limit: 5000

# 工具
/status
/skills list
/permissions
```

---

## 相关文档

- [ARIS README](../README_CN.md) — 完整功能介绍
- [会话恢复指南](../docs/SESSION_RECOVERY_GUIDE_CN.md) — 跨会话状态持久化
- [替代模型组合](../docs/MODELSCOPE_GUIDE.md) — 使用 Kimi/GLM/MiniMax 等
- [工作日志索引](./aris_worklog.md) — 本项目工作记录