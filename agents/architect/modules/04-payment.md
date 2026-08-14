# 模块 04 · 支付（微信 / Apple IAP / Google）

## 范围

Member 域负责订单、会员权益、退款、对账。**服务端是唯一权威**，客户端不直接开通权益。

## 三通道选型

| 通道 | 场景 | 费率 | 合规要点 |
|---|---|---|---|
| **微信支付 V3（小程序）** | 微信小程序内会员/礼物 | 标准 0.6%（年 100 万内享 0.2% 特惠） | 需 ICP 备案 + 营业执照 + 微信开放平台 |
| **Apple IAP** | iOS App 内会员 | 30%（小型开发者 15%；2026 起自动续费首年后续期降至 12%） | 必须走 IAP；绕过会被下架 |
| **Google Play Billing** | Android App | 15%（首年 100 万美元以内）/ 30% | 订阅必须用 Play Billing |

## 会员套餐

参考良配原版：

| 套餐 | 价格 | 期限 | 退款 |
|---|---|---|---|
| 保结婚会员 | ¥2000 | 3 年 | 3 年未领证结婚全额退款 |
| 永久会员 | ¥1000 | 永久 | 不退款 |

**实现建议**：用**连续包月/季/年**而非买断（IAP 友好、收入稳定、避免一次性高额退款风险）。

## 关键设计：掉单防御

```
客户端跳转支付
  ↓
[不立即开通] 等服务端回调
  ↓
  ├─ 5s 内收到回调 → 开通 + 跳转成功页
  └─ 5s 内未收到
      ↓
      启动客户端轮询 (30s 一次 × 5 次)
        ↓
        ├─ 收到 → 开通 + 跳转成功页
        └─ 仍失败 → 进入"待处理"页（人工客服）
```

## 数据模型

```sql
CREATE TABLE orders (
  id              TEXT PRIMARY KEY,           -- 商户订单号
  user_id         BIGINT REFERENCES users(id),
  channel         SMALLINT,                   -- 1 微信, 2 Apple, 3 Google
  product_id      TEXT,
  amount_cents    BIGINT,
  state           SMALLINT,                   -- 0 待支付, 1 已支付, 2 退款, 3 取消
  channel_tx_id   TEXT,                       -- 通道交易号
  created_at      TIMESTAMPTZ,
  paid_at         TIMESTAMPTZ,
  refunded_at     TIMESTAMPTZ
);

CREATE TABLE members (
  user_id         BIGINT PRIMARY KEY REFERENCES users(id),
  tier            SMALLINT,                   -- 1 普通, 2 银, 3 金
  expire_at       TIMESTAMPTZ,
  auto_renew      BOOLEAN DEFAULT TRUE,
  order_id        TEXT
);

CREATE TABLE payment_idempotent (
  channel         SMALLINT,
  channel_tx_id   TEXT,
  order_id        TEXT,
  processed_at    TIMESTAMPTZ,
  PRIMARY KEY (channel, channel_tx_id)
);
```

## 三通道实现细节

### 微信支付 V3

```
下单：商户平台 → POST /v3/pay/transactions/jsapi
  ↓
返回 prepay_id
  ↓
客户端 wx.requestPayment 调起支付
  ↓
回调：POST /api/v1/payment/wechat-callback
  ├─ 验签（RSA, 证书轮换）
  ├─ 幂等表去重
  ├─ 开通会员
  └─ 返回 200
```

### Apple IAP

```
前端：StoreKit 购买 → 获得 JWS receipt
  ↓
后端：Apple Verify Receipt API（生产环境：buy.itunes.apple.com/verifyReceipt）
  ├─ 校验通过
  ├─ 解析订阅事件（6 种）
  ├─ 开通/续期会员
  └─ 返回 200
  ↓
Apple Server Notifications V2（JWS 验签）
  ├─ 订阅更新、取消、退款
  └─ 同步状态
```

### Google Play Billing

```
前端：Google Play Billing 购买 → 获得 purchaseToken
  ↓
后端：Google Play Developer API 验证
  ├─ 校验通过
  ├─ 开通会员
  └─ 返回 200
  ↓
Real-time Developer Notifications (RTDN) via Pub/Sub
  ├─ 订阅更新、取消、退款
  └─ 同步状态
```

## 退款流程

```
用户申请退款（7 天内）
  ↓
客服审核
  ↓
  ├─ 通过
  │   ├─ 微信：调 V3 退款 API
  │   ├─ Apple：调 App Store Server API
  │   └─ Google：调 Google Play Developer API
  │   ↓
  │   关闭会员
  │   ↓
  │   留存退款凭证 ≥ 180 天
  └─ 拒绝：用户可申诉
```

## 对账任务（每日 03:00 Cron）

```
拉取通道昨日所有交易
  ↓
对比 orders 表
  ├─ 一致 → 标记 OK
  └─ 差集
      ├─ 我方有，通道无 → 检查回调日志 + 人工介入
      └─ 通道有，我方无 → 补单 + 告警
```

## API 设计

| Method | Path | 说明 |
|---|---|---|
| POST | `/api/v1/payment/create-order` | 创建订单 |
| GET | `/api/v1/payment/order/:id` | 查询订单状态 |
| POST | `/api/v1/payment/wechat-callback` | 微信回调 |
| POST | `/api/v1/payment/apple-notification` | Apple 通知 |
| POST | `/api/v1/payment/google-notification` | Google RTDN |
| POST | `/api/v1/payment/refund` | 申请退款 |

## 故障与降级

| 故障 | 降级 |
|---|---|
| 微信回调大面积延迟 | 客户端启动轮询补偿 |
| Apple/Google 通知丢失 | 每日主动 Verify Receipt 兜底 |
| 对账差集异常 | 告警 → 人工介入 |
| IAP 退款但未通知 | 客户端主动 Verify Receipt |

## 合规要点

| 合规项 | 要求 |
|---|---|
| **不绕过 IAP** | 任何虚拟会员/礼物必须走 Apple IAP + Google Billing |
| **不引向第三方支付** | 小程序内只能用微信支付 |
| **退款凭证** | 留存 ≥ 180 天 |
| **价格透明** | 必须明确展示价格 + 续期规则 |

## 关联文档

- [架构 · 容器视图](../architecture/02-containers.md)
- [故障 Runbook](../runbook/failure-top10.md)
- [ADR-002 · AI 主供应商](../adr/0002-deepseek-v3.md)