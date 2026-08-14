# ADR-0004 · 数据库选型：PostgreSQL 16 + PostGIS + pgvector

- **Status**: Accepted
- **Date**: 2026-08-14
- **Deciders**: forchain

## Context

社交场景三大刚需：

1. **地理查询**（附近的人）→ PostGIS
2. **向量检索**（AI 分身 / 匹配）→ pgvector
3. **全文检索**（昵称/标签）→ PG tsvector

**约束**：
- 一人开发（不要维护 3 套数据系统）
- 国内云支持好
- 成本可控

## Decision

**采用 PostgreSQL 16 + PostGIS + pgvector，一库三用**。

腾讯云 PG 实例：2C4G（约 ¥220/月）。

## Options Considered

| 选项 | 地理 | 向量 | JSON | 全文 | 评估 |
|---|---|---|---|---|---|
| **PG + PostGIS + pgvector** | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★ | **✅ 采纳** |
| MySQL 9 | ★★★ | ★★ | ★★★ | ★★★ | 备选 |
| PG + Milvus（独立向量库） | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★ | 量大时 |
| PG + ES（独立检索） | ★★★★ | ★★★★ | ★★★ | ★★★★★ | 混合检索 |

## Consequences

### Positive

- **一库三用**：地理 / 向量 / 全文，全在一个数据库里，**省独立向量库运维**
- vector 与业务数据天然 JOIN（不用同步）
- 国内云（腾讯云 / 阿里云）支持好
- JSONB + GIN 索引满足灵活画像存储

### Negative

- pgvector 性能不如 Milvus（亿级向量时差距明显）
- PostGIS 学习曲线
- 大数据量时分区表设计复杂

### Neutral

- 与 Redis 配合（缓存 + DB）

## Review Trigger

1. **向量规模 > 2000 万** → 加 Milvus 共存
2. **QPS > 5000/s** → 评估读写分离 + 分库
3. **混合检索需求变强**（向量+关键词权重调整频繁）→ 加 ES
4. **团队无 PG 经验且只是简单 CRUD** → 退 MySQL

## 关联

- [架构 · 容器视图](../architecture/02-containers.md)
- [模块 · 匹配](../modules/02-matching.md)
- [故障 Runbook](../runbook/failure-top10.md#9-数据库主从延迟)