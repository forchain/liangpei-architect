# ADR-0007 · 拒绝：NestJS 作为后端框架

- **Status**: Rejected
- **Date**: 2026-08-14
- **Deciders**: forchain

## Context

考虑过 NestJS 作为后端框架（模块化、装饰器、TypeScript 一等公民）。

## Decision

**拒绝 NestJS**，采用 Hono（轻量级、API-first）。

## Options Considered

| 框架 | 一人成本 | Serverless 友好 | 学习曲线 | 评估 |
|---|---|---|---|---|
| **Hono** | ★★★★★ | ★★★★★ Web Standard | ★★★★★ 最轻 | **✅ 采纳** |
| Next.js | ★★★★ | ★★★★ | ★★★ | SEO/官网用 |
| **NestJS** | ★★★ | ★★★ | ★★★ 装饰器多 | **❌ 拒绝** |
| Fastify | ★★★ | ★★★ | ★★★ | 自部署 |
| Cloudflare Workers | ★★★ | ★★★★★ | ★★★ | 海外版用 |

## Why Rejected

### 1. 一人开发的模块化收益 < 复杂度

- 模块化（Module）、依赖注入（DI）、装饰器（Decorator）的设计是为了大型团队
- 一人项目里写 `@Injectable() @Module()` 是过度设计
- **Hono 一个文件就能跑**：直接 `app.get('/api', (c) => c.json(...))`

### 2. Serverless 友好度

- NestJS 需要适配 Lambda/Workers，框架本身为长进程设计
- **Hono 直接跑在 Cloudflare Workers / Vercel Edge / Bun**
- 一人最省运维

### 3. 学习曲线

- NestJS 装饰器语法（@Injectable、@Module、@Controller）需要适应
- Hono 直接是 Web Standard API（fetch + Request + Response）

## When to Reconsider

如果出现以下情况，重新评估：

1. **团队扩大到 5+ 人**，模块化价值显现
2. **业务复杂到需要严格分层**（DDD 战术模式）
3. **微服务化**（每个服务独立框架）
4. **合规要求严格审计**（NestJS 的结构化更容易审计）

## 关联

- [架构 · 容器视图](../architecture/02-containers.md)
- [ADR-0001 · 跨端框架](./0001-uniapp-x.md)