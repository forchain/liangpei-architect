# 模块 01 · 用户（注册 / 实名 / 人脸 / 未婚 / 注销）

## 范围

User 域负责用户的身份建立、信任建立、隐私合规。**这是合规高风险模块** —— 婚恋类目必须办齐实名、人脸、未婚三件套。

## 关键流程：注册期 5 步

```
1. 微信登录    → 2. 实名 + 人脸  → 3. 未婚认证  → 4. AI 红娘对话  → 5. 进入匹配池
   ↓                  ↓                  ↓                  ↓
wx.login            人脸核身          承诺书+录像        20min 语音
   ↓                  ↓                  ↓                  ↓
openid+token        留存凭证 ID      法律承诺 5 年      bio_vector
```

## 技术方案

| 环节 | 方案 | 合规要点 |
|---|---|---|
| 微信登录 | `wx.login` → 后端 `auth.code2Session` 换 openid + session_key → 自签 JWT (HS256, 7 天) | token 加密存 Redis；加 refresh |
| 实名 | 腾讯云人脸核身 SDK（实名 + 活体） | 留存核身凭证 ID ≥ 180 天 |
| 未婚 | 民政部接口 / 承诺书 + 视频录像 + AI 语音承诺 | 法律承诺，留存 5 年 |
| 资料画像 | AI 红娘 20min 语音 → 结构化抽取 → bio_vector (1024 维) | 用户隐私协议单独同意 |
| 注销 | 30 天软删 → 物理删除 + 异步清除向量 | 个保法第 47 条 |

## 数据模型（最小集）

```sql
CREATE TABLE users (
  id              BIGSERIAL PRIMARY KEY,
  openid          TEXT UNIQUE NOT NULL,
  unionid         TEXT,
  phone           TEXT,
  real_name_status SMALLINT,        -- 0 未认证, 1 已认证
  marriage_status  SMALLINT,        -- 0 未认证, 1 已认证
  gender          SMALLINT,         -- 1 男, 2 女
  birth           DATE,
  location        GEOGRAPHY(POINT, 4326),  -- PostGIS
  bio_vector      VECTOR(1024),     -- pgvector
  status          SMALLINT DEFAULT 1,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE profiles (
  user_id         BIGINT PRIMARY KEY REFERENCES users(id),
  self_desc       TEXT,
  ideal_partner   JSONB,            -- 结构化择偶画像
  photo_urls      TEXT[],
  verified        BOOLEAN DEFAULT FALSE,
  audit_status    SMALLINT
);

CREATE INDEX idx_users_location ON users USING GIST (location);
CREATE INDEX idx_users_bio_vector ON users USING ivfflat (bio_vector vector_cosine_ops);
```

## API 设计

| Method | Path | 说明 |
|---|---|---|
| POST | `/api/v1/auth/wx-login` | code 换 token |
| GET | `/api/v1/users/me` | 当前用户信息 |
| POST | `/api/v1/users/verify-identity` | 启动人脸核身 |
| POST | `/api/v1/users/verify-marriage` | 提交未婚认证 |
| GET | `/api/v1/users/export` | 数据导出（个保法） |
| DELETE | `/api/v1/users/me` | 注销（30 天后物理删除） |

## 安全要点

- **不要在前端做任何鉴权判断** —— 后端为唯一权威
- **人脸凭证 ID 不能返回给前端** —— 只存后端
- **注销要分软删/硬删** —— 软删 30 天内可恢复，硬删后异步清向量
- **未成年人保护** —— 注册时强制年龄 ≥ 22 岁，否则拦截

## 故障与降级

| 故障 | 降级 |
|---|---|
| 微信登录失败 | 提示"网络异常，请重试"；不上送 Sentry 严重告警 |
| 人脸核身超时 | 允许 3 次重试；3 次失败后人工客服 |
| AI 红娘中断 | 断点续传：会话进度存 Redis，可恢复 |

## 关联文档

- [架构 · 系统上下文](../architecture/01-system-context.md)
- [模块 · 内容审核](./05-audit.md)
- [ADR-005 · AI 编码工具](../adr/0005-cursor-trae.md)