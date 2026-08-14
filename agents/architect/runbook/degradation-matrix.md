# 分层降级开关

> 设计原则：**任何外部依赖故障都不能让整个 App 不可用**。

## 降级矩阵

| 层级 | 正常态 | 降级 1 | 降级 2 | 降级 3 |
|---|---|---|---|---|
| **AI 红娘** | 实时语音 + 流式 LLM | 切文本对话 | 文字 + 选项引导 | "红娘当前繁忙，留言后通知" |
| **AI 分身** | 数字人 + 音色 | 仅文字回复 | 文字 + 模板 | "对方未上线，留言后通知" |
| **AI 军师** | 流式诊断 | 模板话术库 | "军师休息中" | 显示历史建议 |
| **匹配** | 向量召回 + 模型打分 | 向量召回 + 规则 | 按地理 + 标签 | 显示已有 Top-50 |
| **IM** | 环信实时 | 环信异步 + 短信 | 仅留言 | 邮件通知 |
| **审核** | 实时同步 | 异步 + 缓存放行 | 人工抽审 | 全部放行（高风险场景禁用） |
| **支付** | 微信支付 V3 | Apple Pay（iOS） | Google Pay（Android） | 暂停新购 |
| **推送** | APNs + 多通道 | 仅 APNs | 仅站内消息 | 关闭推送 |

## 触发条件

```typescript
// config/degradation.ts
export const degradation = {
  ai_matchmaker: 'normal',    // normal | text_only | template | offline
  ai_avatar: 'normal',
  ai_advisor: 'normal',
  matching: 'normal',
  im: 'normal',
  audit: 'normal',
  payment: 'normal',
  push: 'normal',
};

// 触发降级的判断（伪代码）
function maybeDegrade(metric: string) {
  if (metric === 'llm_error_rate' && llm_error_rate > 0.05) {
    degradation.ai_matchmaker = 'template';
    degradation.ai_avatar = 'text_only';
    degradation.ai_advisor = 'template';
  }
  if (metric === 'im_online_drop' && im_online_drop > 0.3) {
    degradation.im = 'async';
  }
  if (metric === 'audit_latency' && audit_latency > 1.0) {
    degradation.audit = 'async';
  }
}
```

## 恢复策略

| 条件 | 动作 |
|---|---|
| LLM 错误率连续 5min < 2% | 恢复 normal |
| IM 在线人数恢复 | 恢复 normal |
| 队列堆积 < 100 | 恢复 async |

## 客户端适配

降级状态通过 API 返回给前端，前端按状态渲染：

```typescript
// GET /api/v1/system/status
{
  "degradation": {
    "ai_matchmaker": "template",
    "ai_avatar": "text_only",
    "im": "normal"
  },
  "active_provider": "qwen"  // 当前 LLM
}
```

## 降级的成本

- **模板话术库**：质量下降，但用户感知不明显
- **仅文字分身**：核心价值保留，体验打折
- **匹配降级**：用户看到候选变少，**不能完全停**（否则用户流失）

## 关键不降级场景

| 场景 | 不降级 |
|---|---|
| 用户支付 | **永远不降级**（一旦降级，营收受损） |
| 用户注销 | **永远不降级**（合规要求） |
| 数据导出 | **永远不降级**（合规要求） |
| 紧急封禁 | **永远不降级**（合规要求） |

## 后续阅读

- [故障 Top 10](./failure-top10.md)
- [5 分钟值班卡](./incident-response.md)