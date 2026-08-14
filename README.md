# 良配复刻 · 架构与 AI 工程文档

> **项目目标**：独立开发者从 0 到 1 复刻「良配」（深圳良配科技，2026-07 上线）—— 一款 AI 驱动的严肃婚恋匹配小程序 + App，包含 AI 红娘语音对话、AI 分身匿名问答、AI 军师关系诊断、1v1 滑卡匹配、IM、付费会员、内容审核。

> **文档定位**：架构决策 + 模块方案 + AI 辅助开发策略 + 7×24 稳定性预案。**不含可直接运行的应用代码**——本仓库是 design docs + runbook，后续代码在 feature 分支按 issue 拆分。

---

## 快速导航

| 你想知道… | 看这里 |
|---|---|
| 系统长什么样、谁调用谁 | [`docs/architecture/01-system-context.md`](docs/architecture/01-system-context.md) |
| 服务怎么拆分、技术栈选什么 | [`docs/architecture/02-containers.md`](docs/architecture/02-containers.md) |
| AI 红娘/分身/军师怎么工作 | [`docs/architecture/03-components-ai.md`](docs/architecture/03-components-ai.md) |
| 部署在哪里、怎么容灾 | [`docs/architecture/04-deployment.md`](docs/architecture/04-deployment.md) |
| 用户/匹配/IM/支付/审核 怎么做 | [`docs/modules/`](docs/modules/) |
| 用什么 AI 工具开发、怎么交付 | [`docs/ai-dev-strategy/`](docs/ai-dev-strategy/) |
| 出问题怎么办（值班 Runbook） | [`docs/runbook/`](docs/runbook/) |
| 架构决策记录（为什么这样选） | [`docs/adr/`](docs/adr/) |

---

## 一句话技术栈（2026 主推组合）

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

**省钱关键**：DeepSeek 主对话（占 AI 成本 70%）+ pgvector 一库三用（省独立向量库）+ 环信专业版（前 100 DAU 免费）+ 微信珊瑚审核（免费）。

**最贵的不是云，是合规**：M0 就要把算法备案 / 婚介资质 / 深度合成备案办完，否则 M3 上线时被卡。

---

## 关键非功能性指标

| 指标 | 目标 |
|---|---|
| AI 红娘首响 | P95 < 2s |
| IM 消息送达 | P99 < 1s |
| 系统可用性 | 99.9%（DAU 5 万内单可用区） |
| 资料评分 ≥ 70 通过率 | ≥ 80%（创始人基线） |
| 月度运营成本（DAU 1 万） | ¥1,500–2,500 |
| 月度运营成本（DAU 5 万） | ¥8,000–15,000 |

---

## ADR 速查

| # | 决策 | 一句话 |
|---|---|---|
| [001](docs/adr/0001-uniapp-x.md) | 跨端框架 | **uni-app x 蒸汽模式**（一码多端） |
| [002](docs/adr/0002-deepseek-v3.md) | AI 主供应商 | **DeepSeek-V3 + Qwen Embedding** |
| [003](docs/adr/0003-easemob-im.md) | IM | **环信 SDK**（专业版 < 1 万 DAU 免费） |
| [004](docs/adr/0004-postgres-pgvector.md) | 数据库 | **PostgreSQL 16 + PostGIS + pgvector**（一库三用） |
| [005](docs/adr/0005-cursor-trae.md) | AI 编码 | **Cursor Pro + Trae + Claude Code CLI** |
| [006](docs/adr/0006-reject-self-built-im.md) | ❌ 拒绝 | 自建 WebSocket IM（耗 2 个月、消息消失事故高发） |
| [007](docs/adr/0007-reject-nestjs.md) | ❌ 拒绝 | NestJS（一人开发模块化收益 < 复杂度） |

---

## 风险一览

| # | 风险 | 缓解 |
|---|---|---|
| 1 | 算法 / 深度合成备案审核 1–3 个月 | M0 立即启动；同步挂靠主体 |
| 2 | Apple IAP 30% 抽成挤压利润 | 商业模式补足：线下活动 + 真人 1v1 红娘 |
| 3 | DeepSeek 偶发 API 限流 | 多供应商 fallback（DS → 豆包 → Qwen） |
| 4 | 一人维护多栈事故响应慢 | Sentry 必装；关键流程有降级开关 |
| 5 | 婚恋类目审核被拒 | 提前沟通；留"陌生人社交"类目兜底 |
| 6 | AI 红娘对话触发个保法（情感数据） | 授权明确范围；提供"清除情感数据"按钮 |
| 7 | 数字人/音色克隆被滥用 | 严格用户授权 + 合成水印 + 可追溯 |
| 8 | Cursor 2026 自动执行安全漏洞 | 支付/鉴权模块禁用 Agent；强制人工 review |

---

## 交付路线（M0 → M4，约 13 周）

- **M0（第 1–2 周）** 地基 + 合规：注册主体、ICP 备案、算法备案、深度合成备案、工程初始化
- **M1（第 3–6 周）** 核心匹配 + IM：用户体系、滑卡、环信接入、珊瑚审核
- **M2（第 7–10 周）** AI 红娘 + 分身 + 军师：DeepSeek 对话工程、豆包 Realtime、数字人
- **M3（第 11–13 周）** 会员 + 支付 + App 出包：微信支付 V3、IAP、Play Billing、App Store 上架

详细任务清单见 [`docs/ai-dev-strategy/delivery-plan.md`](docs/ai-dev-strategy/delivery-plan.md)。

---

## 许可证

[MIT](LICENSE) © 2026 forchain# liangpei-architect

---

## 调研核实状态（2026-08-14 deep-research 结果）

仓库内的架构、模块、AI 策略、Runbook 均为**复刻设计稿**（一人 0 → 1 落地方案）。下方为已通过 3 票独立来源核实的产品侧事实（来源见 `docs/research/findings.md`）：

| 事实 | 状态 | 来源 |
|---|---|---|
| AI 搜索实验室（性格/学历/工作/MBTI/生活习惯 5 维度） | ✅ 已核实 | 腾讯应用商店描述 |
| AI 分身匿名代问敏感话题（资产/彩礼/健康） | ✅ 已核实 | 36kr + c114 |
| 1v1 匹配机制（一次只与一位聊天） | ✅ 已核实 | 36kr |
| ¥2000「保结婚会员」3 年未领证全额退款 | ✅ 已核实 | 36kr（创始人受访） |
| 2026-07-04 上线 | ⚠️ 仅 1 票（搜狐转载） | 待二次确认 |
| "AI 红娘 20 分钟建档 / 463 字资料 / 82% 完成率" | ❌ 仅单一 36kr 来源 | 视为创始人口述，不入设计稿 |
| LangGraph 7 节点 / MongoDB+ES+ChromaDB 架构 | ❌ 被反驳（来自第三方猜测仓库） | 不采用 |

未在公开来源中找到、属于设计稿的部分：

- AI 红娘的语音对话能力边界（深度调研仅有 1 票提及）
- AI 分身的训练数据来源与一致性保证机制
- AI 军师的具体功能定义（公开来源仅出现名称，无详细描述）
- 系统架构本身（良配未公开技术栈）

**结论**：本仓库的架构设计是合理的"复刻方案"，但不是良配官方的技术披露。任何"对标良配"的市场宣称都需要更多公开信息。
