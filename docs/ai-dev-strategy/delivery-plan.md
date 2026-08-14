# AI 辅助开发 · 交付路线图

> 一人复刻「良配」从 0 到 1 的 13 周交付计划，按里程碑拆 issue，每个 issue 配套 AI 提示词模板。

---

## 总体节奏

```
M0 地基 + 合规   (W1-W2)
M1 核心匹配 + IM (W3-W6)
M2 AI 红娘 / 分身 / 军师 (W7-W10)
M3 会员 + 支付 + App 出包 (W11-W13)
```

> 一人开发的瓶颈是**上下文切换成本**，所以每个 milestone 内部按周分 issue，避免在 5 个技术栈间反复横跳。

---

## M0 · 地基与合规（W1-W2）

**目标**：法律框架 + 工程骨架同时落地，**不写业务代码**。

| Week | Issue | 验证标准 |
|---|---|---|
| W1 | 注册公司主体 + ICP 备案启动 | 工信部系统受理截图 |
| W1 | 申请小程序 AppID + 类目沟通 | 微信审核经理口头确认婚恋可上 |
| W1 | 申请微信支付商户号（婚恋服务类目） | 商户号开通 |
| W1 | 算法备案 + 深度合成备案材料准备 | 模板填写完成，提交 |
| W2 | monorepo 初始化（pnpm + Turborepo） | `pnpm dev` 同时启动 3 个服务 |
| W2 | DB schema 第一版（users / profiles / matches） | `drizzle-kit migrate` 成功 |
| W2 | CI 基础（GitHub Actions：lint + typecheck + test） | PR 必跑绿 |
| W2 | Sentry + CLS + 告警接入 | 触发一次测试告警 |

**AI 工具用法**
- 用 Claude Code 生成 monorepo 脚手架 + tsconfig / eslint / prettier 模板
- 用 DeepSeek-V3 起草算法备案文档（2000 字说明）

---

## M1 · 核心匹配 + IM（W3-W6）

**目标**：用户能注册、看匹配、聊 IM——婚恋 MVP。

| Week | Issue | 验证标准 |
|---|---|---|
| W3 | 微信一键登录 + 手机号绑定 | E2E：扫码 → 登录 → 进入主页 |
| W3 | 资料完善 28 维 + 评分模型 | 单测覆盖评分规则 |
| W4 | 头像上传 + 珊瑚审核 | E2E：上传违规图被拒 |
| W4 | 匹配召回 + 粗排（pgvector） | 给 100 个种子用户跑批，top30 命中率 ≥ 60% |
| W5 | 滑卡 UI + 双向喜欢触发 matches | E2E：A like B → B like A → 进入 IM |
| W5 | 环信账户注册 + 好友关系 | 双方收到匹配成功推送 |
| W6 | IM 消息下发 + 珊瑚审核流水线 | E2E：发违规消息被撤回 |
| W6 | IM 黑名单 + 举报 | 单元测试 + 手动验证 |

**AI 工具用法**
- 用 Cursor Pro 写滑卡 UI（Vue3 + uts）—— 视觉重活
- 用 Trae 写 E2E 测试（Playwright + 微信开发者工具）
- Claude Code 协助写环信回调签名验证（细节多、易错）

---

## M2 · AI 红娘 + 分身 + 军师（W7-W10）

**目标**：三个 AI 能力全部上线，且不依赖单一供应商。

| Week | Issue | 验证标准 |
|---|---|---|
| W7 | DeepSeek-V3 接入 + 多供应商路由 | 故障注入：DS 挂 → 自动切豆包 |
| W7 | AI 红娘对话工程（系统层 + 情景层） | 单测：5 个典型场景 prompt 通过 |
| W8 | 豆包 Realtime 语音 TTS + Whisper ASR | 红娘房间首响 P95 < 2s |
| W8 | AI 红娘 WebSocket 流式协议 | E2E：用户语音 → AI 语音 < 3s |
| W9 | AI 分身训练 + Qwen Embedding | 用户 A 向 B 分身提问，返回符合 B 风格 |
| W9 | AI 军师关系诊断 | E2E：30 条私聊 → 报告卡生成 |
| W10 | 三个 AI 能力的内容安全兜底 | 注入 100 条对抗 prompt，无一绕过 |
| W10 | SLO 仪表盘（首响 / 成功率 / 兜底率） | Grafana 看板上墙 |

**AI 工具用法**
- 用 Claude Code 写 LLM Router（多供应商 fallback 状态机）
- 用 DeepSeek-V3 **自己**生成 Few-shot 评估样本（让 LLM 评估 LLM）
- 用 Cursor 调前端音频播放器的 Opus 解码性能

---

## M3 · 会员 + 支付 + App 出包（W11-W13）

**目标**：能收钱 + 多端覆盖。

| Week | Issue | 验证标准 |
|---|---|---|
| W11 | 微信支付 V3 接入 + 会员开通 | E2E：购买月度会员，权益生效 |
| W11 | 退款 + 对账 | 模拟退款，订单状态正确 |
| W12 | Apple IAP + Google Play Billing | iOS 真机购买成功 |
| W12 | App 打包（uni-app x → iOS / Android） | TestFlight 内测版本通过 |
| W13 | App Store / Google Play 上架 | 审核通过 |
| W13 | 全链路压测（locust 1000 并发） | P95 不退化、错误率 < 0.5% |
| W13 | 7×24 告警演练 + Runbook | P1 故障 30 分钟内定位 |

**AI 工具用法**
- Claude Code 写 IAP 验签 + JWS 解码（容易出错）
- Cursor 调 App 启动性能（首屏 < 1.5s）
- DeepSeek-V3 帮写 ASO 关键词矩阵 + 应用商店描述文案

---

## 每周节奏模板

```
周一 9:00   晨会（自我 review）
周一 10:00  拆本周 issue（用 issue-template.md）
周二~周四   编码 + 测试（Cursor 优先）
周五 14:00  E2E 跑一遍 + Code Review
周五 17:00  周报 + 更新 Runbook（如有新坑）
周六        充电 / 学习新工具
周日        不工作
```

---

## 关键检查点（Gate）

| Gate | 触发 | 不通过则 |
|---|---|---|
| G0 | M0 结束 | 不进入 M1，需先拿到 ICP 备案号 |
| G1 | M1 结束 | 不进入 M2，必须有 100 个种子用户通过冒烟 |
| G2 | M2 结束 | 不进入 M3，AI 内容安全兜底必须 100% 通过 |
| G3 | M3 结束 | 不上线，必须有 7 天灰度数据（崩溃率 < 0.5%） |