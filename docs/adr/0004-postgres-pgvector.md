# ADR-0004 · 数据库选 PostgreSQL 16 + pgvector + PostGIS

| 项 | 值 |
|---|---|
| 状态 | ✅ Accepted |
| 日期 | 2026-08-01 |
| 决策者 | @forchain |

---

## Context

良配数据需求横跨三类：

1. **关系型数据**：用户、订单、订阅、消息
2. **向量数据**：用户画像 embedding、匹配召回
3. **地理数据**：同城匹配、城市过滤

候选方案：

1. PostgreSQL 16 + pgvector + PostGIS（一库三用）
2. PostgreSQL + Milvus（独立向量库）
3. PostgreSQL + Elasticsearch（搜索 + 向量）

---

## Decision

选用**PostgreSQL 16 + pgvector + PostGIS 一库三用**。

### 选型矩阵

| 维度 | 一库三用 | + Milvus | + Elasticsearch |
|---|---|---|---|
| 运维复杂度 | ⭐⭐⭐⭐⭐（一套 PG）| ⭐⭐⭐（两套）| ⭐⭐（三套）|
| 一致性 | ⭐⭐⭐⭐⭐（ACID）| ⭐⭐⭐ | ⭐⭐⭐ |
| 成本（DAU 1 万）| ¥400/月 | ¥800/月 | ¥1,500/月 |
| 向量召回性能 | ⭐⭐⭐⭐（< 50ms）| ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 地理查询 | ⭐⭐⭐⭐⭐（PostGIS）| ❌ | ⭐⭐⭐ |

---

## Consequences

**正面**
- 一套数据库备份 / 恢复 / 监控
- ACID 事务保证匹配池更新与 messages 写入一致
- 节省 ¥400/月（DAU 1 万时）
- pgvector 的 HNSW 索引在 100 万向量内 P95 < 30ms

**负面**
- pgvector 索引维护复杂（VACUUM / REINDEX 频率高）
- 单库在 DAU > 10 万时需分库（已规划升级路径）

---

## Alternatives Considered

### Milvus / Weaviate（独立向量库）
- ❌ 多一套运维
- ❌ 数据双写一致性问题
- ❌ 单独监控告警

### Elasticsearch
- ❌ 主要优势在全文检索，但良配业务全文搜索需求不大
- ❌ 资源消耗大

---

## 关键决策细节

### pgvector 模型选型
- 向量维度：**1024**（Qwen Embedding 输出）
- 索引类型：**HNSW**（构建慢但查询快）
- 距离度量：**cosine**（语义相似度）

### PostGIS 扩展
- 用于同城匹配：`ST_DWithin(geom, ST_MakePoint(lng, lat), distance)`
- 用户表存 `geom GEOMETRY(Point, 4326)` 列，由 lng/lat 同步生成

### Schema 隔离策略
```
public      → 业务表
audit       → 审核相关
im_archive  → 180 天后归档的消息
```

每个 schema 单独授权，便于最小权限。