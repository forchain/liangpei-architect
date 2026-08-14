# Grafana 监控面板（Observability）

> 一人开发的可视化：4 个核心面板 + 关键告警。

## 四个核心面板

### 1. 业务面板（Biz Dashboard）

| 指标 | 数据源 | 用途 |
|---|---|---|
| DAU / WAU / MAU | PG `users.last_active_at` | 健康度 |
| 新注册 | PG `users.created_at` | 增长 |
| 完成红娘率 | PG `ai_sessions` 状态 | 漏斗 |
| 1v1 匹配数 / 解除数 | PG `matches` | 核心 KPI |
| IM 消息量 | 环信 API | 活跃度 |
| 付费转化率 | PG `orders` + `members` | 营收 |
| 退款率 | PG `orders.refunded_at` | 健康 |

```
{
  "panels": [
    { "title": "DAU 趋势", "type": "graph", "query": "SELECT date(last_active_at), count(*) FROM users GROUP BY 1" },
    { "title": "完成红娘率", "type": "stat", "query": "SELECT count(*) FILTER (WHERE state='done')::float / count(*) FROM ai_sessions" },
    { "title": "付费转化率", "type": "gauge", "query": "SELECT count(DISTINCT user_id) FILTER (WHERE state=1) / count(DISTINCT user_id) FROM orders" }
  ]
}
```

### 2. AI 面板（AI Dashboard）

| 指标 | 用途 |
|---|---|
| LLM P50/P95/P99 延迟 | 体验 |
| Token 用量（按供应商） | 成本 |
| 单次成本（按场景） | 成本 |
| 拒答率 | 质量 |
| 越狱拦截数 | 安全 |
| 资料评分分布 | 红娘质量 |
| Match 模型 P95 | 匹配质量 |

```
{
  "panels": [
    { "title": "LLM P95 延迟（按供应商）", "type": "graph" },
    { "title": "Token 用量（每日）", "type": "graph" },
    { "title": "AI 成本（每日 ¥）", "type": "graph" },
    { "title": "拒答率", "type": "stat" },
    { "title": "越狱拦截数", "type": "graph" },
    { "title": "资料评分分布", "type": "histogram" }
  ]
}
```

### 3. 系统面板（System Dashboard）

| 指标 | 用途 |
|---|---|
| API QPS（按路由） | 流量 |
| 错误率（4xx / 5xx） | 稳定 |
| PG 主从延迟 | DB |
| PG 连接数 / 锁等待 | DB |
| Redis 命中率 | 缓存 |
| IM 在线人数 | IM |
| CDN 带宽 | 网络 |

```
{
  "panels": [
    { "title": "API QPS", "type": "graph" },
    { "title": "错误率（4xx / 5xx）", "type": "graph" },
    { "title": "PG 主从延迟", "type": "graph" },
    { "title": "Redis 命中率", "type": "gauge" },
    { "title": "IM 在线人数", "type": "graph" }
  ]
}
```

### 4. 合规面板（Compliance Dashboard）

| 指标 | 用途 |
|---|---|
| 实名通过率 | 合规 |
| 审核命中分布（按场景） | 风险 |
| 封禁数（每日） | 风控 |
| 数据导出/删除请求数 | 隐私 |
| 未婚认证通过率 | 合规 |

```
{
  "panels": [
    { "title": "实名通过率", "type": "gauge" },
    { "title": "审核命中分布", "type": "pie" },
    { "title": "封禁数", "type": "graph" },
    { "title": "数据导出/删除请求", "type": "graph" }
  ]
}
```

## 关键告警规则

```yaml
# alert.rules.yml
groups:
  - name: ai
    rules:
      - alert: LLMHighErrorRate
        expr: sum(rate(llm_errors_total[5m])) / sum(rate(llm_requests_total[5m])) > 0.05
        for: 2m
        annotations:
          severity: critical
          runbook: docs/runbook/failure-top10.md#1-llm-api-全面不可用

      - alert: LLMSlowLatency
        expr: histogram_quantile(0.95, rate(llm_duration_seconds_bucket[5m])) > 3
        for: 5m
        annotations:
          severity: warning
          runbook: docs/runbook/failure-top10.md#5-ai-红娘卡在最后一题

  - name: payment
    rules:
      - alert: PaymentCallbackDelayed
        expr: time() - payment_callback_last_success_timestamp > 300
        for: 1m
        annotations:
          severity: critical
          runbook: docs/runbook/failure-top10.md#4-微信支付回调掉单

  - name: im
    rules:
      - alert: IMDropHigh
        expr: (im_online_now / im_online_1h_ago) < 0.7
        for: 3m
        annotations:
          severity: warning
          runbook: docs/runbook/failure-top10.md#3-im-断线消息丢失
```

## 通知渠道

| 渠道 | 用途 |
|---|---|
| 邮件 | 所有告警 |
| 短信 | 🔴 严重告警 |
| 微信（Server 酱） | 🟠 高 + 🔴 严重 |
| Slack/钉钉（可选） | 团队 |

## 关键仪表盘：单页速览

```
┌─────────────────────────────────────────────────┐
│ 良配 · 系统状态                                   │
├─────────────────────────────────────────────────┤
│ DAU: 1,234  新注册: 56  付费: 12  退款: 1      │
│ LLM: DeepSeek V3 ✓  错误率 0.3%  P95 1.2s      │
│ DB: PG 主从延迟 0.1s ✓                          │
│ IM: 在线 892  跌幅 +2% ✓                        │
│ 队列: audit-queue 12  match-pool 待刷 0 ✓       │
│ 今日成本: ¥234  累计 ¥1,234 / 月预算 ¥3,000    │
└─────────────────────────────────────────────────┘
```

## Sentry 集成

```typescript
import * as Sentry from '@sentry/node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1,  // 10% 采样
  beforeSend(event) {
    // 过滤掉已知噪音
    if (event.exception?.values?.[0]?.type === 'AIProviderTimeout') {
      // 已知的供应商超时，不上报
      return null;
    }
    return event;
  },
});
```

## 关键 SLI / SLO

| SLI | SLO | 测量 |
|---|---|---|
| API 可用性 | 99.9% | 1 - (5xx / total) |
| LLM 首响 | P95 < 2s | llm_duration_seconds |
| IM 消息送达 | P99 < 1s | im_message_latency |
| 支付回调 | P99 < 30s | payment_callback_duration |

## 关联文档

- [故障 Top 10](./failure-top10.md)
- [5 分钟值班卡](./incident-response.md)
- [分层降级开关](./degradation-matrix.md)