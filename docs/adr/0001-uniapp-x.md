# ADR-0001 · 跨端框架选 uni-app x 蒸汽模式

| 项 | 值 |
|---|---|
| 状态 | ✅ Accepted |
| 日期 | 2026-08-01 |
| 决策者 | @forchain |

---

## Context

良配需要覆盖**微信小程序 + iOS + Android** 三端。一人开发团队，无法承担三套独立代码库。

候选方案：

1. **uni-app x 蒸汽模式**（Vue3 + uts）
2. Taro 4（React + TS）
3. 原生三端（Swift / Kotlin / 微信 WXML）

---

## Decision

选用 **uni-app x 蒸汽模式**作为跨端框架。

理由：

| 维度 | 评分 | 说明 |
|---|---|---|
| 生态成熟度 | ⭐⭐⭐⭐⭐ | 700 万开发者，婚恋类案例多 |
| 一码多端 | ⭐⭐⭐⭐⭐ | 同一份 Vue3 代码编译到小程序 + Android + iOS |
| 性能 | ⭐⭐⭐⭐ | 蒸汽模式 uts 已脱离 JS 引擎，比传统 uni-app 快 5 倍 |
| 一人友好度 | ⭐⭐⭐⭐⭐ | Vue3 上手快，AI 工具（Cursor）支持最好 |
| 包体积 | ⭐⭐⭐⭐ | 小程序端控制在 1.5MB 以内 |

---

## Consequences

**正面**
- 一份业务代码，三端运行，开发效率 ×3
- 同一份 TypeScript 类型，前后端共用 `@liangpei/contracts`
- 小程序审核快速（无需等 App Store 审核）

**负面**
- uts 是新语言，AI 工具有时给出错误代码（需人工 review）
- 部分原生能力（如复杂动画）仍需写条件编译
- iOS 包审核偶尔因 uts 运行时被 Apple 警告（需解释 uts 不是热更新）

---

## Alternatives Considered

### Taro 4（React + TS）
- ❌ React 生态对一人开发不够友好，组件库选择多但学习曲线陡
- ❌ Taro 编译到 iOS 性能不如 uts

### 原生三端
- ❌ 一人团队维护成本 ×3，直接否决
- ❌ 不可能 13 周交付

### Flutter
- ❌ 国内生态弱，微信支付 / 珊瑚审核 SDK 适配麻烦
- ❌ 小程序支持不成熟

---

## Notes

- 强制使用 TypeScript + Composition API
- 禁止 `v-if` 与 `v-for` 同元素（性能问题）
- 关键路径必须有原生条件编译兜底（`#ifdef MP-WEIXIN`）