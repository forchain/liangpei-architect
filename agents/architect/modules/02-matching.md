# 模块 02 · 匹配（1v1 / 滑卡 / 召回 / 排序）

## 范围

Match 域是产品核心。**核心约束**：每个用户同时只能与 1 人匹配 — 这是良配最具差异化的设计。解除后 24 小时内不重复推荐同一类人群。

## 匹配三阶段

```
召回 (Recall)  →  粗排 (Pre-rank)  →  精排 (Re-rank)
~500 候选       ~50 候选            ~5 候选 (Top-K)
pgvector         规则 + 地理         Match 模型 7B
地理 PostGIS     互斥过滤            1v1 锁
```

## 技术方案

| 环节 | 方案 | 关键指标 |
|---|---|---|
| 画像向量化 | Qwen3-Embedding / BGE-M3 → pgvector vector(1024) | 写入 P95 < 800ms |
| 预计算匹配池 | 每日 03:00 Cron：每个用户召回 Top-50 候选 → Redis SortedSet | 滑卡首屏 < 200ms |
| 实时打分 | Match 模型 7B（vLLM 部署，FP16 单卡 4090） | P99 < 600ms |
| 地理召回 | PostGIS `ST_DWithin` 50km | 索引 + 分区表 |
| 1v1 锁 | Postgres `SELECT FOR UPDATE` / Redis SETNX | 防并发匹配同一人 |
| 解除 / 重匹配 | 用户主动解除 → 解除事件入库 → 24h 内不重复推荐 | 破冰心理保护 |

## 数据模型

```sql
CREATE TABLE matches (
  id              BIGSERIAL PRIMARY KEY,
  user_a          BIGINT REFERENCES users(id),
  user_b          BIGINT REFERENCES users(id),
  score           REAL,
  state           SMALLINT,         -- 0 待定, 1 已匹配, 2 已解除, 3 已跳过
  vector_distance REAL,
  created_at      TIMESTAMPTZ,
  dissolved_at    TIMESTAMPTZ,
  CHECK (user_a < user_b)         -- 防止重复
);

CREATE TABLE match_locks (
  user_id         BIGINT PRIMARY KEY,
  partner_id      BIGINT,
  locked_at       TIMESTAMPTZ,
  expires_at      TIMESTAMPTZ      -- 默认 30 天
);

CREATE INDEX idx_matches_user_a ON matches(user_a, state);
CREATE INDEX idx_matches_user_b ON matches(user_b, state);
```

## Redis 数据结构

```redis
# 每日 Top-K 候选池（SortedSet）
ZADD match_pool:{user_id} {score} {candidate_user_id}

# 1v1 锁（String + EX）
SET match_lock:{user_id} {partner_id} EX 2592000 NX

# 已解除用户冷却（Set）
SADD dissolved_24h:{user_id} {user_id_xxx}
EXPIRE dissolved_24h:{user_id} 86400
```

## 召回 SQL（简化版）

```sql
WITH candidates AS (
  SELECT id, 1 - (bio_vector <=> $1) AS sim
  FROM users
  WHERE id != $2
    AND status = 1
    AND gender = $3
    AND ST_DWithin(location, $4, 50000)   -- 50km
    AND id NOT IN (SELECT user_b FROM matches WHERE user_a = $2 AND state IN (1,2))
  ORDER BY bio_vector <=> $1
  LIMIT 100
)
SELECT * FROM candidates ORDER BY sim DESC LIMIT 50;
```

## API 设计

| Method | Path | 说明 |
|---|---|---|
| GET | `/api/v1/match/next` | 获取下一组滑卡候选 |
| POST | `/api/v1/match/like/:user_id` | 喜欢 |
| POST | `/api/v1/match/skip/:user_id` | 跳过 |
| POST | `/api/v1/match/dissolve/:user_id` | 主动解除 1v1 |
| GET | `/api/v1/match/current` | 当前 1v1 关系 |

## 性能模型（DAU 1 万假设）

| 操作 | QPS | P95 延迟目标 |
|---|---|---|
| 滑卡 next | 50 | < 200ms |
| like/skip | 30 | < 100ms |
| dissolve | 5 | < 500ms |
| 实时打分（Match 模型） | 10 | < 600ms |

## 关键设计点

1. **预计算 > 实时计算**：Top-K 写 Redis，滑卡走 Redis 而非每次查询 PG
2. **1v1 是产品决策**：通过 `match_locks` 强制约束，不能仅靠前端限制
3. **向量距离用 cosine**：`pgvector` 的 `<=>` 算子
4. **解除事件要可追溯**：保留 audit trail，**不解雇可恢复**

## 故障与降级

| 故障 | 降级 |
|---|---|
| Top-K 池过期 | 触发实时召回；标记降级状态 |
| Match 模型不可用 | 退化为向量召回 + 规则打分 |
| 1v1 锁冲突 | 让用户看到"已被匹配"提示，等待 |
| Redis 不可用 | 降级走 PG（性能下降但功能正常） |

## 关联文档

- [架构 · AI 域组件](../architecture/03-components-ai.md)
- [模块 · IM](./03-im.md)
- [故障 Runbook](../runbook/failure-top10.md)