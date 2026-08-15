# /deep-research 工作流使用指南

> Claude Code 提供的多 agent 深度调研 harness。
> Researcher Agent 的核心工具。

---

## 1. 何时使用

### 适用场景

| 场景 | 说明 |
|---|---|
| 调研一个产品 / 公司 | 「了解一下 XX 公司」「XX App 的技术架构」|
| 调研一个行业 / 赛道 | 「中国 SaaS 市场」「跨境电商竞争格局」|
| 调研一个技术决策 | 「XX vs YY 哪个适合我们的场景」|
| 调研一个法规 / 标准 | 「中国 AI 备案流程」「GDPR 对我们的影响」|
| 调研一个学术 / 文献问题 | 「RAG 的最新进展」「向量数据库对比」|

### 不适用场景

| 场景 | 替代方案 |
|---|---|
| 回答当前会话内的具体问题 | 直接在对话中回答 |
| 写代码 / 修 bug | 使用 `find-docs` / `claude-code-guide` |
| 极简单问题 | 用 WebSearch 直接查 |
| 涉及内部代码 / 知识 | 用 Explore agent |

---

## 2. 触发方式

```
/deep-research <问题>
```

> 如果问题过于宽泛，工作流会先问你 2-3 个澄清问题来收窄范围。

---

## 3. 工作流的 5 个阶段

### Phase 1 · Scope（拆解）

> 把问题拆成 5 个独立搜索角度。
> 典型拆解模板：

```
原始问题：<粘贴>

角度 1: 产品形态与功能
角度 2: 技术架构与选型
角度 3: 商业模式与定价
角度 4: 用户画像与运营数据
角度 5: 监管与合规风险
```

> 角度之间应**正交**——避免重复搜索。

### Phase 2 · Search（并行搜索）

> 5 个 WebSearch agent **并行**执行，每个 agent 负责一个角度。
> 通常每个角度 3-5 次搜索即可。

### Phase 3 · Fetch（去重抓取）

> 汇总 5 个 agent 的搜索结果 → URL 去重 → 抓取 top 15 来源的完整内容。
> 优先抓取一手来源（官网 / 政府 / 财报）> 二手（媒体报道）> 三手（CSDN 博客）。

### Phase 4 · Verify（3 票对抗核查）

> 每条**可证伪声明**（falsifiable claim）走 3 票核查：

```
声明：「X 是 Y」
  ↓
3 个独立 verifier agent 并行评估
  ├─ Verifier A: 我找到 <证据 1>
  ├─ Verifier B: 我找到 <证据 2>  
  └─ Verifier C: 我找到 <反向证据 1>
  
投票规则：
  ≥ 2 票 refute → 标记为 ❌ refuted
  ≥ 2 票 confirm + 无反驳 → 标记为 ✅ verified
  1 票 confirm + 0 反驳 → ⚠️ unverified
  0 票 confirm + ≥ 1 反驳 → ❌ refuted
```

> 详见 `fact-verification-rubric.md`。

### Phase 5 · Synthesize（综合）

> 把所有声明按主题分组 → 按置信度排序 → 生成可读 Markdown 报告 + JSON 原始结果。

---

## 4. 输出物

| 输出 | 用途 |
|---|---|
| `<topic>/README.md` | 调研总览 + 核心结论 |
| `<topic>/findings.json` | 原始声明 + 来源 + 投票详情（机读） |
| `<topic>/fact-verification.md` | 已核实 vs 未核实清单（人读） |
| `<topic>/product-*.md` 等主题文档 | 各维度详细整理 |

---

## 5. 工作流局限

| 局限 | 缓解 |
|---|---|
| 网络搜索被中文防火墙阻挡 | 必要时启用 WebFetch 抓特定 URL |
| 工作流不能访问付费数据库 | 标注为「需付费核实」|
| 不能验证图片 / 视频内容 | 标注为「仅文本可核实」|
| 一次性工作流，**不**长期跟踪 | 重要主题应定期重新调研 |

---

## 6. 实战模板

### 模板 A：调研一个 App

```
/deep-research 请调研「<App 名称>」小程序 / App：
1）核心功能与差异化
2）技术架构推测
3）商业模式与定价
4）用户规模与活跃度
5）监管合规情况
```

### 模板 B：调研一个行业

```
/deep-research 请调研中国「<赛道>」市场：
1）市场规模与增长趋势
2）头部玩家及其差异化
3）主要商业模式与 ARPU
4）监管框架与准入门槛
5）AI / 新技术对该赛道的影响
```

### 模板 C：调研一个技术决策

```
/deep-research 请调研「<技术 A> vs <技术 B>」选型：
1）核心能力对比（基准测试数据）
2）社区与生态成熟度
3）商业支持（SLA / 价格）
4）生产环境案例
5）潜在风险与迁移成本
```

### 模板 D：调研一个法规

```
/deep-research 请调研「<法规名称>」对中国 AI / 婚恋 / 支付 行业的影响：
1）法规原文要点
2）适用对象与豁免
3）合规要求与备案流程
4）处罚案例与执法力度
5）行业内的合规实践
```

---

## 7. 与 Researcher Agent 目录结构的关系

```
agents/researcher/
└── liangpei-product/                ← 工作流输出物
    ├── README.md                    ← 调研总览
    ├── findings.json                ← 工作流原始 JSON
    ├── fact-verification.md         ← 已核实 vs 未核实清单
    ├── product-capabilities.md      ← 产品形态整理
    └── business-model.md            ← 商业模式分析
```

> 每次调研 = 在 `agents/researcher/` 下新建一个子目录，名字对应调研主题。