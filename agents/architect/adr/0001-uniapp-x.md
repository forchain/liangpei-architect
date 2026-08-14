# ADR-0001 · 跨端框架选型：uni-app x（蒸汽模式）

- **Status**: Accepted
- **Date**: 2026-08-14
- **Deciders**: forchain

## Context

需要从 0 到 1 复刻一款 AI 恋爱匹配小程序，覆盖：

- 微信小程序（首发）
- iOS App
- Android App
- （未来）H5 / 支付宝小程序 / 抖音小程序

**约束**：
- 一人开发，AI 辅助
- 同一份代码出多端
- 性能接近原生（不能像 H5 卡顿）
- AI IDE 友好（Vue3/TS 项目结构清晰）

## Decision

**采用 uni-app x（蒸汽模式），Vue3 + uts + Vite 技术栈**。

蒸汽模式（2026 GA）：逻辑层和渲染层都编译为原生（Kotlin/Swift/ArkTS），**无 JS 桥接**，性能等同原生。

## Options Considered

| 选项 | 一码多端 | 性能 | AI IDE 友好 | 评估 |
|---|---|---|---|---|
| **uni-app x** | ★★★★★ | ★★★★★ | ★★★★ | **✅ 采纳** |
| Taro 4 | ★★★★ | ★★★★ | ★★★★★ | 备选（React 背景强时） |
| Remax | ★★★ | ★★★ | ★★★★ | ❌ 仅 React+小程序 |
| 原生 wxml + Swift/Kotlin | ★ | ★★★★★ | ★★ | ❌ 双仓库双语言 |

## Consequences

### Positive

- 一份代码覆盖 6+ 端（小程序/iOS/Android/H5/支付宝/抖音/鸿蒙）
- 性能等同原生（无 JS 桥接损耗）
- Vue3 + uts，AI IDE 模板完整，Cursor/Trae/Claude Code 都能索引
- DCloud 商业背书，长期维护有保障

### Negative

- uts 学习曲线（语法接近 TS，需要适应）
- 平台差异通过条件编译处理，仍需一定适配
- 第三方 SDK 适配可能有 1-2 周延迟

### Neutral

- 包体积比纯 wxml 大 ~15%
- App 端审核仍需走 App Store / Google Play

## Review Trigger

出现以下任一情况，重新评估：

1. **uts 无法调用的小程序原生新 API**（如微信新发布的硬件能力）
2. **App 端需 60fps 复杂动画**（如游戏化匹配）
3. **DCloud 公司停止维护**（观察 GitHub 活跃度）
4. **团队强制 React 背景**（仅当团队扩大时考虑）

## 关联

- [架构 · 容器视图](../architecture/02-containers.md)
- [ADR-0002 · AI 主供应商](./0002-deepseek-v3.md)