# ARIS Agent Usage Active Context

## Latest Update
- Time: 2026-04-29 21:11
- Topic: codex-mainline-deepseek-reviewer
- Request: 判断 Codex 主控、DeepSeek 审稿人是否可运行，并准备后续帮助用户配置运行。
- Findings From Repo:
  - `skills/skills-codex/auto-review-loop-llm/SKILL.md` 已明确支持 DeepSeek provider，base URL 为 `https://api.deepseek.com/v1`，模型包括 `deepseek-chat`、`deepseek-reasoner`。
  - `mcp-servers/llm-chat/server.py` 是通用 OpenAI-compatible reviewer bridge，适合 DeepSeek。
  - 默认 `skills/skills-codex/` 中很多 reviewer-aware skills 使用 secondary Codex agent 或 `mcp__codex__codex`，不能直接调用 DeepSeek；全量替换需要后续改写。
- Implementation Plan:
  - 先部署 `llm-chat` MCP 并注册到 Codex。
  - 先使用 `/auto-review-loop-llm` 验证 DeepSeek reviewer。
  - 如果用户需要完整 ARIS 流水线，再做本地 skills overlay 或批量改写。

## History Summary
- 2026-04-29 21:11: 更新 DeepSeek reviewer 路线和执行断点。
