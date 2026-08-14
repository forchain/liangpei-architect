# ADR-0005 · AI 编码工具：Cursor Pro + Trae + Claude Code CLI

- **Status**: Accepted
- **Date**: 2026-08-14
- **Deciders**: forchain

## Context

一人 + AI 工具链的核心编码工作流。

**约束**：
- 国内访问稳定
- 中文友好
- 安全（避免自动执行漏洞）
- 月成本 < ¥500

## Decision

- **日常编码**：Cursor Pro（$20/月，≈¥145）
- **复杂任务 / 重构**：Claude Code Max（按用量，复杂任务用）
- **国内合规备份**：Trae（字节，免费额度）+ 通义灵码（阿里云栈）

## Options Considered

| 工具 | 价格 | 中文 | 上下文 | Agent | 国内 | 评估 |
|---|---|---|---|---|---|---|
| **Cursor Pro** | $20/月 | ★★★ | 200K | ★★★★★ | ★★★★ | **✅ 日常主力** |
| **Claude Code** | $20-200/月 | ★★★★ | 200K | ★★★★★ | ★★★★ | **✅ 复杂任务** |
| **Trae** | 免费 | ★★★★★ | 200K | ★★★★ | ★★★★★ | **✅ 备份** |
| **通义灵码** | 免费 | ★★★★★ | 128K | ★★★ | ★★★★★ | 备选 |
| Windsurf | $15/月 | ★★★ | 200K | ★★★★ | ★★★ | 备选 |
| GitHub Copilot | $10/月 | ★★★ | 128K | ★★★ | ★★★ | 备选 |
| Codex (OpenAI) | 按 token | ★★ | 256K | ★★★★ | ★★ | 海外网络 |

## Consequences

### Positive

- Cursor 体验最好（Composer、Agent、代码理解）
- Trae 国内合规 + 免费，作为备份
- Claude Code 重度任务（架构重构、bug 排查）
- 三工具覆盖所有场景

### Negative

- **Cursor 2026 Q2 安全报告**：自动执行模式有漏洞 —— 涉及支付密钥、JWT、加密代码必须人工逐行 review
- **Trae 2026 每日对话限额**：重度用户不够用
- **Claude Code 按 token 收费**：重度使用成本高

### Neutral

- 工具切换有学习成本，但风格相似（都是 Claude 系）

## 安全规则（强制）

以下代码 **必须人工逐行 review**，禁用 AI Agent 自动执行：

- 支付密钥、JWT 签发、回调验签
- 数据库迁移、权限变更
- 算法备案、隐私协议文案
- 算法模型 prompt 模板（防止注入）

## Review Trigger

1. **Cursor 安全漏洞修复** → 重新评估 Agent 模式使用范围
2. **Trae 限额放开** → 考虑全切 Trae
3. **新出更好的工具**（如 GPT-5 Codex）→ 重新评估
4. **国内政策变化**（如全面禁用海外 LLM）→ 切国产工具

## 关联

- [AI 辅助开发总策略](../ai-dev-strategy/README.md)
- [AI 工具矩阵](../ai-dev-strategy/tools-matrix.md)