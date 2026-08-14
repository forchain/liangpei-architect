# 🏛️ 架构师 Agent 方案

> **角色**：主架构师 / 平台工程负责人视角
> **交付物**：系统设计、技术选型、ADR、Runbook、AI 工程实践
> **状态**：✅ 已完成（2026-08-14）
> **作者**：forchain（借助 Claude Code + 调研子 Agent）

---

## 本 Agent 视角下的核心判断

1. **业务核心是 AI 体验**：注册期 AI 红娘 + 运行时 AI 分身 + AI 军师 = 三大 Agent
2. **1v1 机制是产品护城河**：同时只能与一人匹配，强制用户认真对待
3. **最贵的不是云，是合规**：M0 启动算法备案 + 婚介资质 + 深度合成备案
4. **省钱关键**：DeepSeek 主对话 + pgvector 一库三用 + 环信专业版免费 + 微信珊瑚免费
5. **稳定性 > 性能**：一人开发，先有降级开关，再谈极致性能

---

## 目录结构

```
agents/architect/
├── architecture/                  # C4 模型 + 部署拓扑
│   ├── 01-system-context.md      # L1 · 系统上下文
│   ├── 02-containers.md          # L2 · 容器视图
│   ├── 03-components-ai.md       # L3 · AI 域组件（红娘/分身/军师）
│   ├── 04-deployment.md          # 部署 + 灾备 + 成本
│   └── diagrams/                 # 4 张 SVG
│       ├── c4-context.svg
│       ├── c4-containers.svg
│       ├── c4-components-ai.svg
│       └── deployment.svg
├── modules/                      # 核心模块技术方案
│   ├── 01-user.md               # 用户/实名/人脸/未婚/注销
│   ├── 02-matching.md           # 1v1 匹配 + 召回 + 排序
│   ├── 03-im.md                 # 环信 IM + AI 军师 + 心有灵犀
│   ├── 04-payment.md            # 微信 V3 + Apple IAP + Play Billing
│   └── 05-audit.md              # 珊瑚 + 阿里云 + AI 注入防御
├── ai-dev-strategy/              # AI 辅助开发策略
│   ├── README.md                # 3 条铁律 + 工作流
│   ├── tools-matrix.md          # IDE / LLM / UI / 测试工具对比
│   └── delivery-plan.md         # M0-M4 共 13 周交付清单
├── runbook/                      # 7×24 稳定性预案
│   ├── failure-top10.md         # Top 10 故障 + 防御 + 5min 处置
│   ├── degradation-matrix.md    # 分层降级开关
│   ├── incident-response.md     # 值班 Runbook
│   └── observability.md         # Grafana 4 面板 + 告警
└── adr/                          # 架构决策记录
    ├── 0001-uniapp-x.md         # ✅ 跨端：uni-app x 蒸汽模式
    ├── 0002-deepseek-v3.md      # ✅ AI 主供应商：DeepSeek-V3 + Qwen
    ├── 0003-easemob-im.md       # ✅ IM：环信 SDK
    ├── 0004-postgres-pgvector.md # ✅ DB：PG + PostGIS + pgvector
    ├── 0005-cursor-trae.md      # ✅ AI 编码：Cursor + Trae
    ├── 0006-reject-self-built-im.md  # ❌ 拒绝自建 WebSocket
    └── 0007-reject-nestjs.md    # ❌ 拒绝 NestJS
```

---

## 关键交付统计

- **总文件数**：28（4 SVG + 23 MD + 1 README）
- **总行数**：~3057
- **ADR**：5 采纳 + 2 拒绝（含 Review Trigger）
- **故障预案**：Top 10 + 降级开关 + 值班卡 + 监控面板
- **覆盖视角**：架构 + 模块 + 开发策略 + 稳定性 + 决策

---

## 关键 ADR 速查

| # | 决策 | 主推 | 切换条件 |
|---|---|---|---|
| 0001 | 跨端框架 | uni-app x 蒸汽模式 | 出现 uts 无法调用的 API |
| 0002 | AI 主供应商 | DeepSeek-V3 + Qwen Embedding | DeepSeek 重大事故 / 中文细腻度极致 |
| 0003 | IM | 环信 SDK（< 1 万 DAU 免费） | 深度微信生态需求 |
| 0004 | 数据库 | PG 16 + PostGIS + pgvector | 向量规模 > 2000 万 |
| 0005 | AI 编码 | Cursor Pro + Trae + Claude Code | 国内政策 / 新工具发布 |

---

## 一句话技术栈

```
uni-app x (Vue3 + uts 蒸汽模式)
  ↓
Hono on 腾讯云轻量 4C8G  ←→  PostgreSQL 16 + PostGIS + pgvector
  ↓                              ↓
环信 IM SDK                DeepSeek-V3 + Qwen Embedding
  ↓
微信珊瑚内容审核 + 阿里云兜底
  ↓
微信支付 V3 / Apple IAP / Google Play Billing
```

**月成本估算（DAU 1 万）**：¥3,000–5,000

---

## 与其他 Agent 的协作点

| Agent | 架构师依赖 | 架构师提供 |
|---|---|---|
| 产品分析师 | — | 业务边界定义、技术约束（如 IAP 30% 抽成） |
| 合规法务 | 个保法要求 → 影响 DB 设计 | 算法备案号、深度合成备案号 |
| 财务顾问 | 单次 AI 成本（¥0.1-2）→ 商业模式 ROI | 月度云成本估算（¥1.5k-15k） |
| UI/UX 设计师 | 性能预算（P95 < 2s）→ 动效规范 | 状态机、降级时的 UI 表现 |
| QA 工程师 | 故障 Top 10 → 测试用例 | 评估集、关键路径指标 |
| 增长营销 | 上线 Checklist → 推广时机 | 用户增长对架构的压力测试 |

---

## 快速入口

- [整体技术栈与一句话总结](#一句话技术栈)
- [C4 架构图入口 →](./architecture/01-system-context.md)
- [故障 Top 10 入口 →](./runbook/failure-top10.md)
- [ADR 索引 →](./adr/0001-uniapp-x.md)
- [M0-M4 13 周交付计划 →](./ai-dev-strategy/delivery-plan.md)

---

**作者**：forchain · **最后更新**：2026-08-14 · **许可证**：MIT