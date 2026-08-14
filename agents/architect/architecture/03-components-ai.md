# 03 · C4 Level 3 — AI 域组件级（红娘 / 分身 / 军师）

## 范围

深入 AI 域内部的三个 Agent（红娘 / 分身 / 军师）以及共享能力层。它们都依赖同一套底层（LLM 路由、Token 计量、Prompt 缓存、注入检测、JSON Schema 强约束、流式 SSE 输出）。

## 图

![AI Components](./diagrams/c4-components-ai.svg)

## AI 红娘（Matchmaker Agent）· 注册期

**触发**：用户完成微信登录 + 人脸核身 + 未婚认证后

**目标**：20 分钟语音对话 → 抽取结构化画像 → 资料用心度评分 ≥ 70 才放行

| 子组件 | 职责 | 备注 |
|---|---|---|
| 语音采集 | `wx.getRecorderManager` 采集 16k PCM | 客户端 |
| ASR | 语音转文字 | 豆包 ASR |
| Prompt 引擎 | System Prompt + 历史 + 工具 | 模板版本化 |
| LLM 主对话 | 生成对话与追问 | DeepSeek-V3 |
| 结构化抽取 | 对话结束 → JSON 画像 | DeepSeek-V3，response_format: json_object |
| TTS | 文字转语音播放 | 豆包 TTS |

**输出**：
- `profiles.self_desc`（自画像文字）
- `profiles.ideal_partner`（择偶画像）
- `users.bio_vector`（1024 维向量，pgvector）
- `ai_sessions`（完整对话存档，用于后续优化）

## AI 分身（Avatar Agent）· 运行时

**触发**：他人向你的"分身"提问（通过匿名中间人机制）

**目标**：化解敏感问题（彩礼、房贷、家庭、感情史）—— 由 AI 替用户作答，必要时匿名转给真人

| 子组件 | 职责 | 备注 |
|---|---|---|
| 用户提问 | 文本输入 | — |
| 意图识别 | 分类：可答 / 不可答 / 敏感转真人 | DeepSeek-V3 |
| 资料检索 (RAG) | pgvector 向量检索相关 chunks | Qwen Embedding |
| 数字人形象 | 火山数字人 API | 用户授权 + 厂商预置形象 |
| 匿名转发 | 不可答时转给真人 | 保留隐私 |
| 用户音色克隆 | 用户上传 30s 样本 | 必须单独授权书 |

**风险**：
- 数字人形象合规 → 只用厂商预置形象 + 合成水印
- 音色滥用 → 用户单独授权书 + 90 天留存

## AI 军师（Advisor Agent）· 运行时

**触发**：用户问"对方是否对我有意""冷场聊什么""对方忙还是放弃？"

**目标**：基于双方画像 + 聊天记录 → 给出好感度判断 + 破冰话术

| 子组件 | 职责 | 备注 |
|---|---|---|
| 双方画像 | 从 PG 读取 | — |
| 聊天记录 | 滑动窗口 + 摘要 | LLM 摘要压缩 |
| LLM 推理 | 关系诊断 | DeepSeek-R1（强逻辑） |
| 好感度判断 | 输出 0-100 分数 | JSON Schema |
| 破冰话术 | 3 条候选话术 | JSON Schema |

**彩蛋**：双方 24h 内都问"对方是否对我有意"→ 触发"心有灵犀"成就 → 双向解锁。

## 共享能力层

被三个 Agent 共用，避免重复造轮子：

| 能力 | 实现 |
|---|---|
| 多 LLM 路由 | config/llm.json 切换 active_provider，支持 fallback |
| Token 计量 | 每次调用记录 tokens_used、cost_cny、按日聚合 |
| Prompt 缓存 | 相同 query+context 5 分钟内复用 Redis |
| 注入检测 | 用户输入前置关键词 + 嵌入相似度双判 |
| JSON Schema 强约束 | LLM 输出必须通过 Zod 校验，失败重试 1 次后告警 |
| 流式输出 | SSE / EventSource，前端首响 < 1s |

## 一次 AI 红娘会话的数据流

```
1. wx.login() → 后端用 auth.code2Session 换 openid + 自签 JWT
2. wx.startFacialRecognitionVerify() → 腾讯云人脸核身
3. wx.getRecorderManager.start() → 采集 16k PCM
4. WebSocket 上行音频 → ASR → Prompt 引擎 → LLM 流式 → TTS → 下行音频
5. 每轮结果写入 ai_sessions.messages (jsonb)
6. 会话结束 → 结构化抽取 → 写入 profiles + bio_vector
7. 评分 ≥ 70 → 推送"资料已上线"事件 → 进入匹配池
```

## AI 域常见故障 & 防御

| 故障 | 触发 | 防御 |
|---|---|---|
| ASR 静默丢音 | 网络抖动 / 用户静音 | 客户端检测静音 > 3s 自动提示；服务端超时 8s 重连 |
| LLM 超时 | 供应商限流 | 30s 心跳 + 自动续写 token + 多供应商 fallback |
| Prompt 注入 | 恶意用户在自画像/聊天塞指令 | 前置审核 + LLM 重写层 + JSON Schema |
| 数字人合规 | 自训练真人形象 | 只用厂商预置形象 + 合成水印 |
| 音色滥用 | 未经授权克隆 | 用户单独授权书 + 90 天留存 + 可撤回 |
| AI 拒答 | 命中红线 | 多供应商 fallback + 模板话术兜底 |
| 情感越界 | 输出违反政策 | 系统提示词红线 + 输出 JSON Schema + 人工抽审 1% |

## 后续阅读

- 部署拓扑 → [04-deployment.md](./04-deployment.md)
- AI 工具链 → [../ai-dev-strategy/README.md](../ai-dev-strategy/README.md)
- 故障 Runbook → [../runbook/failure-top10.md](../runbook/failure-top10.md)