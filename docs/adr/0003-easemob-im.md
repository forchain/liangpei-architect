# ADR-0003 · IM 选环信专业版

| 项 | 值 |
|---|---|
| 状态 | ✅ Accepted |
| 日期 | 2026-08-01 |
| 决策者 | @forchain |

---

## Context

良配核心场景是 1v1 私聊，需要稳定的 IM 服务。一人开发无能力维护 WebSocket 长连接集群。

候选方案：

1. 环信 IM（专业版）
2. 腾讯云 IM
3. 阿里云 IMS
4. 自建 WebSocket + MongoDB

---

## Decision

选用**环信 IM 专业版**。

### 选型矩阵

| 维度 | 环信专业版 | 腾讯云 IM | 阿里云 IMS | 自建 |
|---|---|---|---|---|
| 价格（DAU 1 万）| ¥0（< 100 免费）| ¥1,800/月 | ¥2,000/月 | ¥800/月 |
| 婚恋类案例 | 多 | 一般 | 少 | 无 |
| SDK 稳定性 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | - |
| 一周接入 | ✅ | ✅ | ✅ | ❌（2 月+）|
| 消息送达 P99 | < 1s | < 1s | < 1.5s | 不可控 |

---

## Consequences

**正面**
- 前 100 DAU 完全免费，0 → 100 DAU 阶段无 IM 成本
- WebHook 回调签名验证清晰，5 行代码搞定
- 婚恋行业案例多（陌陌、百合都基于环信）

**负面**
- 锁定环信生态，未来迁移成本中等
- 偶尔 BGP 故障（已准备 local fallback，详见 ADR-006）

---

## Alternatives Considered

### 腾讯云 IM
- ❌ 价格高 6 倍（DAU 1 万时）
- ❌ 控制台不如环信直观

### 阿里云 IMS
- ❌ 价格最高
- ❌ 婚恋类案例少

### 自建（详见 ADR-006）
- ❌ 已正式拒绝

---

## 关键集成点

| 集成项 | 实现 |
|---|---|
| 账户注册 | 服务端 HTTP `POST /users` |
| 好友关系 | 服务端 `POST /contacts/users/{username}/users/{friendName}` |
| 消息下发 | 客户端 SDK 直发，WebHook 回调做审核 |
| 推送 | 极光代理 APNs / FCM |
| 消息撤回 | 服务端 `DELETE /messages/{msgId}` |

> 注意：环信账号体系与良配账号体系是**双轨**——用 `easemob_uid = liangpei_uid` 做映射。