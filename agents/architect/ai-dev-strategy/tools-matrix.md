# AI 工具矩阵

## 编码 IDE

| 工具 | 价格 | 中文友好 | uni-app 友好 | 上下文 | Agent | 推荐度 |
|---|---|---|---|---|---|---|
| **Cursor Pro** | $20/月 | ★★★ | ★★★★ | 200K | ★★★★★ | ★★★★★ 日常编码主力 |
| **Claude Code** | $20-200/月 | ★★★★ | ★★★★ | 200K | ★★★★★ | ★★★★★ 复杂任务 |
| **Trae（字节）** | 免费 | ★★★★★ | ★★★★ | 200K | ★★★★ | ★★★★ 国内首选 |
| **通义灵码** | 个人免费 | ★★★★★ | ★★★★ | 128K | ★★★ | ★★★★ 阿里云栈 |
| Windsurf | $15/月 | ★★★ | ★★★ | 200K | ★★★★ | ★★★ |
| GitHub Copilot | $10/月 | ★★★ | ★★★ | 128K | ★★★ | ★★★ |

**2026 关键变化**（影响选型）：
- Cursor 2026 Q2 出现"打开仓库自动执行"安全漏洞 — **敏感代码慎用 Agent 模式**
- 阿里 2026 Q2 全公司禁用 Claude Code（环境信息收集）— **国内企业合规优先 Trae/通义灵码**
- Trae 2026 设置每日对话限额 — **重度用户建议 Cursor Pro + 国内 LLM**

**推荐组合**：
- 日常编码：**Cursor Pro（$20/月）**
- 复杂架构：**Claude Code Max**（按用量）
- 国内合规备份：**Trae + 通义灵码**

## LLM API

| 模型 | 输入 | 输出 | 上下文 | 综合 |
|---|---|---|---|---|
| **DeepSeek-V3** | ¥2/M | ¥8/M | 64K | ★★★★★ 主对话默认 |
| DeepSeek-R1 | ¥4/M | ¥16/M | 64K | ★★★★★ 深度推理 |
| DeepSeek V4 Flash | $0.14/M | $0.28/M | 1M | ★★★★ 长上下文 |
| Qwen3-Max | ¥2/M | ¥6/M | 1M | ★★★★★ 备用 |
| Qwen3-Plus | ¥0.32-0.96/M | ¥1.28-3.84/M | 1M | ★★★★ 中等 |
| Qwen3-Flash | ¥0.03-0.2/M | ¥0.13-0.8/M | 1M | ★★★★ 大规模粗排 |
| 豆包 2.1 Pro | ¥6/M | ¥30/M | 256K | ★★★★ 字节生态 |
| 豆包 2.1 Turbo | ¥3/M | ¥15/M | 256K | ★★★★ 高性价比 |
| GLM-5.2 | ¥8/M | ¥28/M | 1M | ★★★ 长上下文 |
| Kimi K3 | $3/M | $15/M | 1M | ★★★ 贵 |

**推荐组合**：
- 主对话：**DeepSeek-V3**
- 推理 / 军师：**DeepSeek-R1**
- Embedding：**Qwen3-Embedding 或 BGE-M3**
- 长文档：**DeepSeek V4 Flash**（¥1/¥2 每 M）

## UI 设计 → 代码

| 工具 | 用途 | 价格 |
|---|---|---|
| **v0.dev (Vercel)** | 单页 UI 草图 → React/Vue | 免费 + Pro $20/月 |
| **Figma Make** | 设计 → 代码（端到端） | Figma 全家桶 |
| **Figma MCP Server** | 设计稿喂给 Cursor | 免费 |
| Lovable | 整站 + Supabase 后端 | $25/月起 |
| Bolt.new | 全栈出原型，1M tokens 免费 | 免费 + Pro |
| Replit Agent | 全栈 Agent | $25/月 |

**推荐流程**：
1. **Figma 设计** → 用 Figma Make 出一稿
2. **AI 生成 UI** → v0 出 React/Vue 单页组件
3. **接入 MCP** → Cursor 通过 Figma MCP 直接读设计稿
4. **拼装** → 把 v0 组件搬到 uni-app x 工程里

## 自动化测试

| 工具 | 场景 | 推荐度 |
|---|---|---|
| **Playwright** | 小程序 WebView、E2E（Web/管理后台） | ★★★★★ |
| **Maestro**（YAML） | 移动端 E2E，最简 | ★★★★★ 一人复刻首选 |
| Appium | 原生 App UI 测试 | ★★★ |
| minitest/wechat-miniprogram-automator | 小程序自动化 | ★★★★ |
| **Vitest** | 单元测试 | ★★★★★ |
| k6 | 压测 | ★★★★★ |

**推荐组合**：Maestro（移动端主流程）+ Playwright（Web/管理后台）+ Vitest（单元）+ k6（压测）。

## 监控与日志

| 工具 | 用途 | 价格 |
|---|---|---|
| **Sentry** | 前端 + 后端异常 | 免费版 |
| **Prometheus** + Grafana | 指标 + 看板 | 开源自托管 |
| 阿里云日志服务 | 日志聚合 | 按量 |
| Better Stack | 一体化监控 | $20/月起 |

## 关联文档

- [AI 辅助开发总策略](./README.md)
- [交付计划 M0–M4](./delivery-plan.md)