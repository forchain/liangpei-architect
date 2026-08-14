# ADR-0007 · ❌ 拒绝 NestJS 作为后端框架

| 项 | 值 |
|---|---|
| 状态 | ✅ Accepted（拒绝方案） |
| 日期 | 2026-08-01 |
| 决策者 | @forchain |

---

## Context

评估后端框架时，认真考虑过 NestJS。这份 ADR 是为了**正式记录拒绝原因**。

---

## Decision

**正式拒绝 NestJS**。选用 Hono（详见 ADR 主架构文档）。

---

## 拒绝原因

### 1. 一人开发的模块化收益 < 复杂度

| NestJS 概念 | 对一人项目的价值 |
|---|---|
| Module / Controller / Service | ❌ 一个 service 文件即可，模块拆分是给团队用的 |
| DI 容器 | ❌ 直接 `new` 更直观 |
| Decorator 反射 | ❌ 增加心智负担，AI 工具（Cursor）经常写错 |
| Guards / Pipes / Interceptors | ❌ 一个 middleware 函数即可 |
| Microservices | ❌ 一人项目用不到微服务 |

NestJS 解决的是**5+ 人团队 + 长生命周期大型应用**的痛点。一人 13 周交付，引入 NestJS 反而是负担。

### 2. AI 工具友好度

| 框架 | Cursor 生成准确率 |
|---|---|
| Hono | ⭐⭐⭐⭐⭐（直接 handler 函数） |
| Express | ⭐⭐⭐⭐⭐ |
| NestJS | ⭐⭐⭐（AI 常写出错的装饰器 / 模块导入） |

Cursor 在 Hono / Express 上生成代码几乎不需要修改，在 NestJS 上经常需要返工。

### 3. 启动性能

```
Hono cold start:  15ms
NestJS cold start: 380ms
```

一人部署在腾讯云轻量 4C8G 上，Hono 能省更多内存给 AI 服务。

---

## Consequences

**正面**
- 心智简单：handler → service → DB，三层足够
- AI 工具友好
- 启动快、内存低
- TS 一等公民

**负面**
- 缺少内置大型项目结构（需要自律）
- 没有官方企业级插件（但社区 Hono 中间件已丰富）

---

## Alternatives Considered

### Express
- ✅ 也可以选，生态最大
- ❌ TS 支持不如 Hono 现代
- 备选：若 Hono 团队出现严重问题，回退到 Express

### Fastify
- ✅ 性能更好
- ❌ 生态不如 Hono / Express

### Elysia / tRPC
- ❌ 生态太新，第三方 SDK 兼容性不确定

---

## 触发重评估的条件

| 条件 | 行动 |
|---|---|
| 团队扩到 3 人以上 | 评估迁移到 NestJS |
| 单仓代码量 > 5 万行 | 评估模块化方案 |

---

## Hono 的关键实践

```typescript
// src/index.ts — 简洁够用
import { Hono } from 'hono';
import { logger } from 'hono/logger';
import { userRouter } from './routes/user';
import { matchRouter } from './routes/match';

const app = new Hono();

app.use('*', logger());
app.use('*', authMiddleware());  // 全局鉴权

app.route('/v1/user', userRouter);
app.route('/v1/match', matchRouter);

app.get('/health', (c) => c.json({ ok: true }));

export default app;
```

每条路由就是一个文件：

```typescript
// src/routes/match.ts
import { Hono } from 'hono';
import { zValidator } from '@hono/zod-validator';
import { z } from 'zod';

export const matchRouter = new Hono()
  .get('/feed', zValidator('query', z.object({ cursor: z.string().optional() })),
    async (c) => {
      const { cursor } = c.req.valid('query');
      const list = await matchService.getFeed(c.get('userId'), cursor);
      return c.json(list);
    })
  .post('/action', zValidator('json', z.object({
    candidateId: z.number(),
    action: z.enum(['skip', 'like', 'super']),
  })), async (c) => {
    // ...
  });
```

---

## 经验教训

> **框架复杂度应该匹配团队规模**。
> NestJS 是给 5+ 人团队设计的，一人项目用 Hono 是更优解。