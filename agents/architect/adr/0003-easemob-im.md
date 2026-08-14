# ADR-0003 · IM 选型：环信 SDK + UIKit

- **Status**: Accepted
- **Date**: 2026-08-14
- **Deciders**: forchain

## Context

需要支持 1v1 私聊场景的 IM 系统。

**约束**：
- 跨端覆盖（小程序 + iOS + Android）
- 消息可靠（不丢失、不重复）
- 离线推送（多通道）
- 一人开发（不要自己写断线重连）

## Decision

**采用环信 IM SDK + UIKit（专业版 < 1 万 DAU 免费）**。

## Options Considered

| 选项 | 一人成本 | 跨端 | 价格 | 消息可靠 | 评估 |
|---|---|---|---|---|---|
| **环信 IM** | ★★★★★ | ★★★★★ | ★★★★★（<1万免费） | ★★★★★ | **✅ 采纳** |
| 腾讯云 IM | ★★★★★ | ★★★★★ | ★★★★ | ★★★★★ | 备选 |
| 融云 IM | ★★★★ | ★★★★★ | ★★★★ | ★★★★★ | 出海备选 |
| 自建 WebSocket | ★ | ★★ | ★★★★★ | ★★ | ❌ 拒绝（见 ADR-006） |

## Consequences

### Positive

- SDK 一体化最成熟，断线重连 / ACK / 漫游 / 多端同步全免
- UIKit 一键集成，节省 60% 前端工作量
- 专业版 < 1 万 DAU 免费（满足 MVP 阶段）
- HarmonyOS / uni-app 覆盖完整

### Negative

- 环信控制台在国内偶尔访问慢
- 高级功能（如阅后即焚）需企业版
- 与微信原生 IM 互通能力有限（但本项目无需）

### Neutral

- 消息存储在环信云（不存我方 PG）—— 仅冗余索引

## Review Trigger

1. **环信出现重大稳定性事故**（> 4h 服务中断）
2. **深度微信生态集成需求**（如小程序客服消息互通）→ 切腾讯云 IM
3. **DAU > 1 万**，专业版不再免费 → 评估成本 vs 切腾讯云
4. **极端出海需求** → 切融云 IM

## 关联

- [模块 · IM](../modules/03-im.md)
- [ADR-0006 · 拒绝自建 IM](./0006-reject-self-built-im.md)
- [故障 Runbook](../runbook/failure-top10.md#3-im-断线消息丢失)