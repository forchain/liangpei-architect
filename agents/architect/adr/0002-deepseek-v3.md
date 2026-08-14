# ADR-0002 · AI 主供应商选型：DeepSeek-V3 + Qwen Embedding

- **Status**: Accepted
- **Date**: 2026-08-14
- **Deciders**: forchain

## Context

需要为 AI 红娘（语音对话）、AI 军师（推理）、AI 分身（问答）三个 Agent 选择主供应商。

**约束**：
- 中文为主，细腻度要求高
- Function Calling 必须稳定
- 单价要低（AI 成本占总成本 50%+）
- 国内合规（不能出境）

## Decision

- **主对话**：DeepSeek-V3（¥2 / ¥8 每 M tokens）
- **推理 / 军师**：DeepSeek-R1（¥4 / ¥16 每 M）
- **Embedding**：Qwen3-Embedding 或 BGE-M3
- **Fallback**：豆包 2.1 Pro → Qwen3-Max

## Options Considered

| 模型 | 输入 | 输出 | 中文 | FC | 稳定性 | 综合 |
|---|---|---|---|---|---|---|
| **DeepSeek-V3** | ¥2 | ¥8 | ★★★★★ | ★★★★★ | ★★★★★ | **✅ 采纳** |
| Qwen3-Max | ¥2 | ¥6 | ★★★★★ | ★★★★★ | ★★★★★ | 备选 |
| 豆包 2.1 Pro | ¥6 | ¥30 | ★★★★ | ★★★★ | ★★★★★ | 字节生态 |
| GLM-5.2 | ¥8 | ¥28 | ★★★★ | ★★★★ | ★★★★ | 长上下文 |
| Kimi K3 | $3 | $15 | ★★★★ | ★★★ | ★★★★ | 贵 |

## Consequences

### Positive

- DeepSeek-V3 价格最低（¥2/¥8）+ 中文质量最佳 + Function Calling 强
- 占 AI 成本 70% 场景最优
- 与 Kimi、豆包、Qwen fallback 链路清晰
- 国内合规（深圳创业 + 微信生态）

### Negative

- DeepSeek 偶发限流，需要 fallback 链路
- 长上下文不如 Qwen（64K vs 1M）
- 部分中文文学性表达不如豆包

### Neutral

- 需自建 LLM 路由层（AI Gateway）

## Review Trigger

1. **DeepSeek 出现重大稳定性事故**（连续 24h 错误率 > 10%）→ 切豆包
2. **中文细腻度要求极致**（如高端情感咨询）→ 切 Qwen3-Max
3. **长上下文需求 > 64K**（读 1v1 整本聊天记录）→ 切 DeepSeek V4 Flash
4. **价格大幅上涨** → 切 Qwen3-Flash（¥0.03-0.2 / ¥0.13-0.8）

## 关联

- [架构 · AI 域组件](../architecture/03-components-ai.md)
- [模块 · IM](../modules/03-im.md)
- [故障 Runbook](../runbook/failure-top10.md#1-llm-api-全面不可用)