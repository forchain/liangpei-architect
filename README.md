# 良配复刻 · 多 Agent 方案聚合站

> **本仓库** = 「良配」（深圳良配科技 / 创始人曾歆勋，前 Kimi 搜索负责人）从 0 到 1 复刻所需的多视角方案集合。
> 每个 Agent 提交自己的方案到独立子目录，本 README 作为导航页。

---

## 🗂️ Agent 方案索引

| Agent | 视角 | 状态 | 入口 |
|---|---|---|---|
| 🏛️ **架构师**（architect） | 系统架构 / 技术选型 / ADR / Runbook | ✅ 已交付 | [→ agents/architect/README.md](agents/architect/README.md) |
| 📊 产品分析师 | 用户画像 / 竞品 / 商业模型 | ⏳ 待交付 | — |
| ⚖️ 合规法务 | 个保法 / 算法备案 / 婚介资质 / 数字人合规 | ⏳ 待交付 | — |
| 💰 财务顾问 | 单用户成本 / 营收模型 / 退款承诺风险 | ⏳ 待交付 | — |
| 🎨 UI/UX 设计师 | 交互流程 / 视觉规范 / 设计 tokens | ⏳ 待交付 | — |
| 🧪 QA 工程师 | 测试策略 / 用例库 / 评估集 | ⏳ 待交付 | — |
| 📣 增长营销 | 上线推广 / KOL / ASO / 私域 | ⏳ 待交付 | — |

> **添加新 Agent**：在 `agents/<agent-name>/` 下创建独立目录 + `README.md` 入口；在本表追加一行。

---

## 🏛️ 架构师方案（agents/architect/）

> 主仓库首发由架构师 Agent 提供：从 0 到 1 的系统设计、技术选型、AI 工程实践与稳定性预案。

### 快速导航

| 你想知道… | 看这里 |
|---|---|
| 系统长什么样、谁调用谁 | [architecture/01-system-context.md](agents/architect/architecture/01-system-context.md) |
| 服务怎么拆分、技术栈选什么 | [architecture/02-containers.md](agents/architect/architecture/02-containers.md) |
| AI 红娘/分身/军师怎么工作 | [architecture/03-components-ai.md](agents/architect/architecture/03-components-ai.md) |
| 部署在哪里、怎么容灾 | [architecture/04-deployment.md](agents/architect/architecture/04-deployment.md) |
| 用户/匹配/IM/支付/审核 怎么做 | [modules/](agents/architect/modules/) |
| 用什么 AI 工具开发、怎么交付 | [ai-dev-strategy/](agents/architect/ai-dev-strategy/) |
| 出问题怎么办（值班 Runbook） | [runbook/](agents/architect/runbook/) |
| 架构决策记录（为什么这样选） | [adr/](agents/architect/adr/) |

### 一句话技术栈（2026 主推）

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

### 关键非功能性指标

| 指标 | 目标 |
|---|---|
| AI 红娘首响 | P95 < 2s |
| IM 消息送达 | P99 < 1s |
| 系统可用性 | 99.9%（DAU 5 万内单可用区） |
| 月度运营成本（DAU 1 万） | ¥3,000–5,000 |

---

## 🚦 全局状态

| 维度 | 状态 |
|---|---|
| 架构文档 | ✅ 完整（28 文件 / ~3000 行） |
| ADR 决策 | ✅ 5 采纳 + 2 拒绝 |
| 合规清单 | 🟡 M0 阶段（需启动算法备案 / 婚介资质） |
| 工程代码 | 🔴 未启动（按 issue 拆分） |
| 上线时间 | 📅 M3 末（约 13 周后） |

---

## 🤝 多 Agent 协作约定

1. **每个 Agent 一个目录**：`agents/<agent-name>/`，互不污染
2. **每个目录一个 README**：作为该 Agent 方案的入口页
3. **跨 Agent 引用**：用相对路径 `../<other-agent>/...`
4. **冲突处理**：两个 Agent 矛盾时，在各自 README 里写"分歧点"+ 触发 ADR 讨论
5. **统一术语**：见 [`agents/architect/architecture/`](agents/architect/architecture/) 中定义的概念图

---

## 📜 许可证

[MIT](LICENSE) © 2026 forchain

---

## 🔗 快速链接

- GitHub: <https://github.com/forchain/liangpei-architect>
- 当前 PR: [#1 docs: bootstrap architecture & engineering playbook](https://github.com/forchain/liangpei-architect/pull/1)