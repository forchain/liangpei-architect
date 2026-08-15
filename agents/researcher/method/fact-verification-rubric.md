# 事实核查 SOP（3 票对抗核查）

> deep-research 工作流的 Phase 4 详细 SOP。
> Researcher Agent 内部质量控制的核心。

---

## 1. 核查对象：可证伪声明（Falsifiable Claim）

> **只核查可证伪的声明**——主观意见 / 未来预测 / 个人偏好不核查。

| 类型 | 是否核查 | 示例 |
|---|---|---|
| 事实陈述 | ✅ 核查 | 「良配 2026-07-04 上线」 |
| 数据数字 | ✅ 核查 | 「月活 800 万」|
| 因果关系 | ✅ 核查 | 「因为 X 导致 Y」 |
| 功能存在性 | ✅ 核查 | 「支持 AI 红娘对话」|
| 主观评价 | ❌ 不核查 | 「界面很好看」 |
| 未来预测 | ❌ 不核查 | 「明年会增长 50%」 |
| 个人偏好 | ❌ 不核查 | 「X 比 Y 好用」|

---

## 2. 3 票核查流程

```
声明 D：「<具体内容>」
  ↓
Step 1: 构造查找 query（针对声明）
  ↓
Step 2: 3 个独立 verifier agent 并行搜索
  ├─ Verifier 1: 寻找印证证据
  ├─ Verifier 2: 寻找印证证据
  └─ Verifier 3: 寻找反驳证据
  ↓
Step 3: 收集 votes + evidence
  ↓
Step 4: 投票规则判定
```

---

## 3. 投票规则

| 状态 | 条件 | 含义 |
|---|---|---|
| ✅ **verified** | ≥ 2 confirm + 0 refute | 事实可信 |
| ⚠️ **unverified** | 1 confirm + 0 refute | 仅 1 票，不足以下定论 |
| ❌ **refuted** | ≥ 1 refute（推翻 confirm）| 反向证据强 |
| ❌ **refuted** | 0 confirm + ≥ 1 refute | 缺乏支持 + 有反驳 |
| ⚠️ **contested** | ≥ 1 confirm + ≥ 1 refute | 存在争议，需人工仲裁 |

### 投票矩阵速查

```
              Confirm 数
              0       1       2       3
Refute 0  | ?       ⚠️       ✅       ✅
Refute 1  | ❌      ⚠️       ⚠️       ⚠️
Refute 2  | ❌      ❌       ❌       ❌
Refute 3  | ❌      ❌       ❌       ❌
```

---

## 4. 证据收集要求

### 必需字段

| 字段 | 说明 |
|---|---|
| `source_url` | 引用 URL |
| `source_type` | 一手 / 二手 / 三手 |
| `source_quality` | 可靠 / 一般 / 不可靠 |
| `quote` | 原文摘录（避免转述失真）|
| `date_accessed` | 访问日期 |

### 可信度判断

| 来源类型 | 默认权重 |
|---|---|
| 政府 / 监管文件 | 高 |
| 公司官方公告 | 高 |
| 公司财报（经审计）| 高 |
| 主流媒体（央媒 / 36kr / 财新等）| 中 |
| 行业研究报告（艾瑞 / QuestMobile）| 中 |
| 个人博客 / CSDN | 低 |
| 论坛 / 知乎匿名回答 | 极低 |

> 详见 `source-quality-tier.md`。

---

## 5. 常见陷阱

### 陷阱 1：同一来源反复印证

> 36kr 文章被 5 个媒体转载，但**只是同一来源**——不算 5 票独立印证。

```
❌ 错误：
  36kr 报道 X
  搜狐转载 36kr X
  c114 转载 36kr X
  → 看似 3 票，实际只有 1 票

✅ 正确：
  36kr 报道 X
  c114 独立采访 X
  腾讯应用商店 App 描述显示 X
  → 3 票独立印证
```

### 陷阱 2：创始人口述 = 事实？

> 「据创始人介绍，月活 50 万」≠ 「月活 50 万」——口述需要独立验证。

```
创始人 / CEO 在专访中说 X → ⚠️ unverified（创始人立场可能带营销色彩）
CEO 说 X + 第三方数据印证 X → ✅ verified
```

### 陷阱 3：技术架构的「合理推测」≠ 事实

> 「良配很可能使用 uni-app」≠ 「良配使用 uni-app」——技术架构无官方披露就是 unverified。

```
「合理推测 X」 → ⚠️ unverified，明确标注「推测」字样
```

### 陷阱 4：过去 vs 现在的混淆

> 2023 年的报道描述的功能 ≠ 2026 年的功能。

```
2023 年报道：「平台有 AI 红娘」 → 仅验证 2023 年的事实
2026 年现在：「平台有 AI 红娘」 → 需重新验证
```

---

## 6. 输出格式

每条声明在 findings.json 中的标准格式：

```json
{
  "claim": "良配 2026-07-04 上线",
  "category": "产品-发布",
  "votes": {
    "confirm": 1,
    "refute": 0
  },
  "verdict": "unverified",
  "confidence": "low",
  "sources": [
    {
      "url": "https://m.sohu.com/a/1052826795_421354",
      "type": "secondary",
      "quality": "medium",
      "quote": "良配 AI 婚恋小程序于 7 月 4 日上线",
      "date_accessed": "2026-08-14"
    }
  ],
  "notes": "仅 1 票（搜狐转载），无其他独立来源印证"
}
```

---

## 7. 人工仲裁（争议情况）

> 当一条声明 `contested`（同时有 confirm 和 refute）时，需人工仲裁：

```
Step 1: 阅读所有证据全文
Step 2: 判断证据权重（一手 > 二手 > 三手）
Step 3: 判断时效性（最新证据优先）
Step 4: 决定 verdict（✅ / ❌ / ⚠️）
Step 5: 在 findings.json 中记录仲裁理由
```

> **原则**：一手 + 最新 = 决定性证据。

---

## 8. 一句话原则

> **「只有 2 个独立来源印证的事实才标 ✅，其他一律标 ⚠️ 或 ❌」**
> ——这是 Researcher Agent 的质量底线。