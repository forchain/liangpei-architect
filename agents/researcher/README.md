# 🔬 调研员 Agent 方案

> **角色**：深度调研员 / 事实核查员 / 文献综述作者
> **交付物**：良配产品调研档案、竞品技术栈分析、行业背景、调研方法论
> **状态**：✅ 首次交付（2026-08-15）
> **作者**：forchain（借助 /deep-research 工作流）

---

## 本 Agent 的边界（与产品分析师区分）

| 维度 | 产品分析师 | 调研员（我） |
|---|---|---|
| 关注 | PRD、用户画像、用户故事 | 客观事实、文献、数据 |
| 产出 | 功能清单 + 优先级 | 调研报告 + 来源引用 + 核实结论 |
| 方法 | 用户访谈、可用性测试 | deep-research 工作流（搜索 → 抓取 → 3 票核查） |
| 立场 | 主观判断（应该做什么） | 客观陈述（事实是什么） |

> 我的本职是**降低团队的决策风险**——任何一个产品 / 商业判断都必须能追溯到我整理的事实。

---

## 本 Agent 视角下的核心判断

1. **「AI 红娘 20 分钟建档 / 463 字资料 / 82% 完成率」是创始人口述，未独立验证**——任何产品决策不应以此为基线
2. **¥2000 保结婚会员的退款机制是真实验证过的创新**——3 票核查通过（36kr + c114 + 创始人公开口径），且符合深圳样本口径
3. **AI 婚恋不是新赛道**：Soul 与伊对早已探索，但「AI 介入严肃婚恋决策」良配是首家
4. **婚介 + AI 服务双重监管**——任何方案都必须考虑个保法 + 算法备案 + 深度合成备案三层叠加

---

## 目录结构

```
agents/researcher/
├── README.md                          # 本入口
├── liangpei-product/                  # 良配产品专项调研
│   ├── README.md                      # 调研全景与核实状态
│   ├── findings.json                  # deep-research 工作流原始结果
│   ├── fact-verification.md           # 已核实 vs 未核实事实清单
│   ├── product-capabilities.md        # AI 红娘/分身/军师产品形态
│   └── business-model.md              # ¥2000 保结婚会员 + 退款机制
├── competitive-landscape/             # 竞品分析
│   ├── README.md
│   ├── direct-competitors.md           # 百合/陌陌/Soul/伊对 等
│   └── ai-differentiators.md          # 与非 AI 婚恋产品的差异化
├── industry-context/                  # 行业背景
│   ├── README.md
│   ├── chinese-dating-market.md       # 中国婚恋市场规模与趋势
│   └── regulatory-environment.md      # 婚介 + AI 备案双重监管
└── method/                            # 调研方法论
    ├── README.md
    ├── deep-research-workflow.md      # 工作流使用指南
    ├── fact-verification-rubric.md    # 3 票对抗核查 SOP
    └── source-quality-tier.md         # 一手 / 二手 / 不可靠分级
```

---

## 关键交付统计

- **总文件数**：13（11 MD + 1 JSON + 1 README）
- **调研方法**：deep-research 工作流（5 角度并行搜索 + 3 票对抗核查）
- **核实事实**：7 条（4 条已核实 / 3 条被反驳 / 5 条仅 1 票未独立确认）
- **覆盖视角**：良配产品 + 竞品 + 行业 + 监管 + 方法论

---

## 与其他 Agent 的协作点

| Agent | 调研员依赖 | 调研员提供 |
|---|---|---|
| 🏛️ 架构师 | — | 良配公开技术栈（如有）作为架构方案的对标参考 |
| 📊 产品分析师 | 调研员整理的「事实 vs 推测」清单 | PRD 写作时的事实基础 |
| ⚖️ 合规法务 | 监管背景文档 | 算法备案、深度合成备案的政策原文 |
| 💰 财务顾问 | 商业模型 + 退款承诺的潜在风险 | 行业营收数据、用户付费意愿调研 |
| 🎨 UI/UX 设计师 | 竞品交互对比 | — |
| 🧪 QA 工程师 | 已反驳事实清单 | 测试用例中的边界场景 |
| 📣 增长营销 | 行业趋势 + 用户画像 | 投放市场细分、ASO 关键词验证 |

---

## 快速入口

- [良配产品调研入口 →](./liangpei-product/README.md)
- [竞品分析入口 →](./competitive-landscape/README.md)
- [行业背景入口 →](./industry-context/README.md)
- [调研方法论入口 →](./method/README.md)

---

**作者**：forchain · **最后更新**：2026-08-15 · **许可证**：MIT