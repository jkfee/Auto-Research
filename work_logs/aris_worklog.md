# ARIS 工作流 - 合并工作日志

> 本文件合并所有分散的 `*_active_context.md` 文件，形成统一的工作记录。

---

## 文件说明

| 文件 | 说明 |
|------|------|
| `aris_worklog.md` | 对话历史和工作记录（主记录） |
| `file_changelog.md` | 代码和配置文件修改记录 |
| `aris_usage_guide.md` | ARIS 使用指南（用户手册） |

---

## 最新状态（2026-05-05）

### API 连通性验证

| 服务 | 状态 | 详情 |
|------|------|------|
| **w.ciykj.cn (GPT-5.4)** | ✅ 正常 | HTTP 200, 响应正常 |
| **DeepSeek (deepseek-v4-pro)** | ✅ 正常 | HTTP 200, 响应正常 |
| **llm-chat MCP server** | ✅ 正常 | 初始化成功，处理请求正常 |

### 当前配置

| 组件 | 配置 |
|------|------|
| **Codex CLI** | `w.ciykj.cn` + `gpt-5.5` (主) / `gpt-5.4` (reviewer), xhigh |
| **ARIS executor** | `gpt-5.4` @ `w.ciykj.cn/v1` |
| **ARIS reviewer** | `deepseek-v4-pro` @ `api.deepseek.com` |
| **ARIS-Code** | v0.4.4 @ `/data1/user/fangjiukai/.local/bin/aris` |

---

## 历史工作记录

### 2026-04-30 02:45 — DeepSeek reviewer key routing fix

**主题**: 修复 ARIS-Code LlmReview 误用 executor key 导致 DeepSeek reviewer 401。

**问题**: LlmReview 把 executor 的 w.ciykj key 当成 DeepSeek key 用了。

**解决方案**:
- 修改 `/data1/user/fangjiukai/.local/bin/aris`:
  - `EXECUTOR_API_KEY` 保留 executor 网关 key
  - `OPENAI_API_KEY` 强制由 DeepSeek `LLM_API_KEY` 注入
  - `ARIS_REVIEWER_AUTH_TOKEN` 强制由 DeepSeek `LLM_API_KEY` 注入
- 修改 `/data1/user/fangjiukai/.config/aris/aris.env`:
  - 移除 `OPENAI_API_KEY` 行，避免 executor key 污染 reviewer
- 修改两个 `auto-review-loop-llm/SKILL.md`:
  - DeepSeek base URL 改为 `https://api.deepseek.com`
  - 模型改为 `deepseek-v4-pro` / `deepseek-v4-flash`

**验证**: LlmReview smoke test 成功返回内容，未再出现 DeepSeek 401。

---

### 2026-04-30 01:25 — ARIS workspace-write permission default

**主题**: ARIS 会话默认 read-only，无法写入 `review-stage/` 报告文件。

**解决方案**:
- 修改 `/data1/user/fangjiukai/.local/bin/aris`，默认追加 `--permission-mode workspace-write`
- 保留 ARIS executor/reviewer env 覆盖逻辑

**验证**: 默认启动状态显示 `Permissions workspace-write`，可写入 `review-stage/`。

---

### 2026-04-30 00:53 — ARIS executor w.ciykj GPT-5.4

**主题**: 将 ARIS-Code executor 切换到 `https://w.ciykj.cn/v1`，模型为 `gpt-5.4`。

**解决方案**:
- 新增 `/data1/user/fangjiukai/.config/aris/aris.env`，存放 executor 专用环境变量
- 修改 `/data1/user/fangjiukai/.local/bin/aris`，支持读取 `aris.env` 覆盖 executor
- 更新 `/data1/user/fangjiukai/.config/aris/config.json`，executor 为 `gpt-5.4`

**注意**: executor key 与 w.ciykj 网关可能不匹配，需验证。

---

### 2026-04-30 00:41 — ARIS DeepSeek base URL 修正

**主题**: DeepSeek 官方 OpenAI-compatible 调用 base_url 应为 `https://api.deepseek.com`。

**解决方案**:
- 修改 wrapper 中的 `ARIS_REVIEWER_BASE_URL` 从 `https://api.deepseek.com/v1` 修正为 `https://api.deepseek.com`
- 保持 `EXECUTOR_BASE_URL=https://api.deepseek.com` 与 `deepseek-v4-pro` 默认模型

---

### 2026-04-30 00:30 — ARIS-Code CLI standalone install

**主题**: 安装 ARIS-Code v0.4.4 独立 CLI，使用 DeepSeek wrapper。

**解决方案**:
- 安装 ARIS-Code v0.4.4 到 `/data1/user/fangjiukai/.local/share/aris-code/v0.4.4/aris`
- 创建 wrapper `/data1/user/fangjiukai/.local/bin/aris`，自动读取 `llm-chat/.env`
- 创建 `/data1/user/fangjiukai/.local/bin/aris-real` 指向原始二进制

---

### 2026-04-30 00:07 — DeepSeek-only Codex config attempt

**主题**: 尝试将 Codex CLI 切换到 DeepSeek-only 配置。

**结果**:
- Codex CLI 0.126 拒绝 `wire_api = "chat"`，需要 `wire_api = "responses"`
- DeepSeek `/responses` 和 `/v1/responses` 返回 404，需要 Chat-to-Responses 适配

---

### 2026-04-29 23:56 — w.ciykj.cn GPT-5.4 Codex config

**主题**: 切换 Codex 主控模型到 `gpt-5.4` with `xhigh` reasoning。

**配置**:
- model: `gpt-5.4`
- base_url: `https://w.ciykj.cn`
- wire_api: `responses`
- reasoning effort: `xhigh`

---

### 2026-04-29 23:35 — Official OpenAI Codex config

**主题**: 将 Codex CLI 切换到官方 OpenAI API。

**配置**:
- provider: `OpenAI`
- base_url: `https://api.openai.com/v1`
- wire_api: `responses`
- requires_openai_auth: `true`

---

### 2026-04-29 23:02 — Codex stream disconnect diagnosis

**主题**: Codex + GPT-5.5 配置下流式响应断开。

**分析**: 使用 `w.ciykj.cn/gpt-5.5` 时请求到达 Responses streaming path，但在 `response.completed` 前反复断开。

---

## 配置变更历史

详见 `file_changelog.md`

---

## 关键文件路径

```
~/.codex/config.toml               # Codex CLI 配置
~/.codex/auth.json                 # Codex API 认证
~/.codex/mcp-servers/llm-chat/    # DeepSeek MCP server
~/.config/aris/config.json         # ARIS executor/reviewer 配置
~/.config/aris/aris.env            # ARIS executor 环境变量
~/.local/bin/aris                  # ARIS wrapper 启动脚本
/data1/user/fangjiukai/Auto-claude-code-research-in-sleep/  # ARIS 工作流源码
```

---

## 使用建议

### 启动 ARIS

```bash
# 在项目目录下
cd /data1/user/fangjiukai/Pointcept-main
/data1/user/fangjiukai/.local/bin/aris

# 或使用 tmux 长期运行
tmux new -s aris
cd /data1/user/fangjiukai/Pointcept-main
/data1/user/fangjiukai/.local/bin/aris
```

### 检查状态

```bash
# 在 ARIS 中
/status
/mcp list
```

---

## 后续工作

- 验证 w.ciykj.cn executor key 是否有效
- 继续 ViT 随机冻结实验（见 `/data1/user/fangjiukai/vision/work_logs/`）
- 使用 `/auto-review-loop` 进行论文 review