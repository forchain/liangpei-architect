# ADR-0005 · AI 编码工具组合：Cursor + Trae + Claude Code

| 项 | 值 |
|---|---|
| 状态 | ✅ Accepted |
| 日期 | 2026-08-01 |
| 决策者 | @forchain |

---

## Context

一人复刻良配的核心杠杆是 AI 工具。需要选定每个工具的角色边界，避免工具之间打架。

候选工具：

| 工具 | 优势 | 劣势 |
|---|---|---|
| Cursor Pro | 全仓库上下文、Composer、UI 强 | 决策深度一般 |
| Trae | E2E 测试专家、调试 UI 强 | 业务代码生成一般 |
| Claude Code CLI | 思考深度最高、可执行命令 | UI 微调慢 |
| GitHub Copilot | 行内补全 | 无全局视角 |
| Windsurf | 类似 Cursor | 生态不如 Cursor |

---

## Decision

**Cursor Pro（主力前端）+ Trae（测试）+ Claude Code CLI（架构 / 文档 / 重构）** 三层组合。

### 角色分工

```
┌─────────────────────────────────┐
│  Cursor Pro                     │  ← 前端 UI、组件、样式
│  模型：Sonnet                   │
└─────────────────────────────────┘
┌─────────────────────────────────┐
│  Trae                           │  ← E2E、Playwright、调试
│  模型：Sonnet                   │
└─────────────────────────────────┘
┌─────────────────────────────────┐
│  Claude Code CLI                │  ← 架构、ADR、重构、复杂 bug
│  模型：Sonnet（默认）/ Opus（关键决策）│
└─────────────────────────────────┘
```

---

## Consequences

**正面**
- 每个工具都在最擅长的场景
- Cursor 改 UI 不用等 CLI 启动
- Claude Code 处理架构级改动不会污染前端工作

**负面**
- 3 个工具月费合计 ¥200/月
- 需要在工具间切换上下文（用 git worktree 隔离）

---

## 安全护栏（一人开发特别重要）

### `.cursorrules` 强制配置

```yaml
mode: manual                    # 禁用自动执行
require_human_review:
  - "**/payment/**"            # 支付模块必须人工
  - "**/auth/**"               # 鉴权必须人工
  - "**/moderation/**"         # 审核必须人工
forbidden_paths:
  - ".env*"
  - "infra/terraform/**"
```

### Claude Code 工作流

1. 改动 > 200 行 → 先 ExitPlanMode → 写 plan → 等批准
2. 涉及支付 / 鉴权 → 不允许 Agent 自动执行，必须 human-in-loop
3. 遇到 bug → 先 `superpowers:systematic-debugging` 再动手

---

## Alternatives Considered

### 全用 Cursor
- ❌ Cursor 改架构级代码风险高（容易引入大改）
- ❌ E2E 调试不如 Trae

### 全用 Claude Code
- ❌ UI 工作响应慢（CLI 不直接编辑 diff 视图）
- ❌ 月费是 Cursor 的 4 倍

### 用 Cody / Continue
- ❌ 生态不如 Cursor
- ❌ Composer 模式 Cursor 独有

---

## 成本对比

| 工具 | 月费 | 主要价值 |
|---|---|---|
| Cursor Pro | $20 | UI + 全局上下文 |
| Trae | ¥0（公测） | E2E |
| Claude Code Pro | $20 | 架构 + 文档 |
| 合计 | $40/月 ≈ ¥290/月 | 人月节省 ≥ 5 天 |

> ROI：1 天节省 > 月费。强推。