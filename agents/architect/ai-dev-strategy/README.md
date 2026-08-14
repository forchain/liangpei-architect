# AI 辅助开发总策略

> **目标**：一人 + AI 工具链，从 0 到 1 在 13 周内交付一个生产级 AI 恋爱匹配小程序。

## 三条铁律

### 1. 骨架先生成，细节自己补

Cursor Composer 出项目骨架 → 人工 review 关键模块（支付、鉴权、加密）→ 局部用 Agent 补功能。

### 2. 关键代码禁用 Agent 自动执行

**Cursor 2026 Q2 安全报告**指出自动执行有漏洞 —— 涉及以下代码必须人工逐行 review：

- 支付密钥、JWT 签发、回调验签
- 数据库迁移、权限变更
- 算法备案、隐私协议文案

### 3. AI 生成代码 100% 加测试

AI 最容易写出"看起来对、边界全错"的代码 —— **边界用例必须人工补**。

## 工具矩阵速览

| 环节 | 主推 | 备份 | 月成本 |
|---|---|---|---|
| 编码 IDE | Cursor Pro | Trae / Claude Code CLI | ¥145 |
| LLM API | DeepSeek-V3 + Qwen Embedding | 豆包 / Kimi | ¥500–2k |
| UI 设计 → 代码 | v0.dev + Figma Make | Lovable / Bolt.new | ¥0–150 |
| 单元测试 | Vitest | Jest | 免费 |
| E2E（Web） | Playwright | Cypress | 免费 |
| E2E（移动端） | Maestro | Appium | 免费 |
| 异常监控 | Sentry 免费版 | 自建日志 | 免费 |

详细见 [`tools-matrix.md`](./tools-matrix.md)。

## 工作流（每日节奏）

```
09:00  看 Sentry 昨日异常（5 min）
09:15  GitHub Issues / 本地 TODO 排序
09:30  Cursor 写主代码（支付/AI prompt/匹配逻辑）
12:00  Lunch
13:00  Cursor + Claude Code 复杂任务（架构重构）
15:00  v0.dev 出 UI 草图 → 拼装到 uni-app x
16:00  Maestro 跑移动端 E2E
17:00  Vitest 单元测试补边界
18:00  Commit + push + 更新 TODO
```

## 详细交付计划

见 [`delivery-plan.md`](./delivery-plan.md)（M0–M4 共 13 周）。

## 关键 ADR

| 决策 | 文档 |
|---|---|
| 跨端框架 | [ADR-001](../adr/0001-uniapp-x.md) |
| AI 主供应商 | [ADR-002](../adr/0002-deepseek-v3.md) |
| AI 编码工具 | [ADR-005](../adr/0005-cursor-trae.md) |