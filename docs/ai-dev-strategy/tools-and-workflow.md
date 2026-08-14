# AI 辅助开发 · 工具与流程

> 一人复刻良配，AI 工具不是装饰——是工程倍增器。本文档定义每个工具的边界与协作模式。

---

## 工具矩阵

| 工具 | 用途 | 何时用 | 何时不用 |
|---|---|---|---|
| **Cursor Pro** | 前端编码、UI 实现、组件打磨 | 写 Vue3 / uts / CSS / 动画 | 涉及支付/鉴权的服务端代码 |
| **Trae** | 测试用例、E2E 脚本 | 写 Playwright 测试 | 业务逻辑实现 |
| **Claude Code CLI** | 全栈规划、重构、文档、复杂 bug | 重构、写 ADR、排查疑难 | UI 微调（响应慢） |
| **DeepSeek-V3** | 代码生成、SQL 优化、文档翻译 | 写常规 CRUD、翻译 README | 需要联网搜索最新 API 时 |
| **GitHub Copilot** | 行内补全、注释转代码 | 写重复样板 | 关键决策逻辑 |

---

## 分层使用策略

### 第 1 层：Cursor（重前端）

**优势**：
- 上下文看到整个 repo
- Composer 模式适合大改 UI
- Cmd+K 单点修改响应快

**使用守则**：
1. **始终用 Sonnet 模型**——Opus 在 UI 工作上溢价不显著
2. **Composer 前先 Figma / 手画草图**——别让 AI 瞎猜视觉
3. **每次 Composer 后必须人工 review diff**——Cursor 会引入不必要的 import
4. **禁用 Composer 自动执行**——`.cursorrules` 强制 `mode: manual`

### 第 2 层：Trae（重测试）

**优势**：
- 专门做 E2E，调试 UI 比 Cursor 强
- 截图 / 录像 / 失败重试都内置

**使用守则**：
1. 每个 issue 必带 1 个 E2E 测试
2. 测试代码 100% 自己写——别让 AI 写测试
3. flaky test 必须 quarantine，不允许阻塞 CI

### 第 3 层：Claude Code CLI（重规划与重构）

**优势**：
- 思考深度最高，适合大改
- 可以执行命令、跑测试、看日志
- 可以写 ADR、设计架构

**使用守则**：
1. **永远在 main 分支之外的 worktree 跑**——避免污染
2. **改动 > 200 行必须 plan first**——先 ExitPlanMode 再动手
4. **遇到 bug 必先 systematic-debugging skill**——不靠感觉改
5. **复杂任务用并行 subagents**——独立模块并行写

### 第 4 层：DeepSeek-V3（重杂活）

**优势**：
- 中文语义理解好、价格便宜
- 写 CRUD、文档翻译性价比高

**使用守则**：
1. 仅在 Cursor / Claude Code 都不擅长的场景用
2. 输出必须人工 review，不能直接 commit

---

## `.cursorrules` 模板

```yaml
# 强制规则（一人开发的安全护栏）
mode: manual                # 禁用自动执行
auto_apply_diffs: false     # 改之前必须让人确认

# 安全模块
forbidden_paths:
  - "**/payment/**"
  - "**/auth/**"
  - "**/moderation/**"
require_human_review:
  - "*.env*"
  - "infra/**"
  - "migrations/**"

# 代码风格
max_line_length: 100
prefer_functional: true
forbid_default_export: true
test_framework: vitest
```

---

## Git 工作流

```
main (受保护)
  └─ feat/liangpei-XXX-issueN (feature)
       └─ chore/liangpei-XXX-subtask (可选)
```

| 分支类型 | 创建 | 合并 | 部署 |
|---|---|---|---|
| `feat/*` | 从 main | squash 到 main | 自动 staging |
| `fix/*` | 从 main | squash 到 main | 自动 staging |
| `hotfix/*` | 从 main | merge（非 squash）| 立即 production |

**commit message 模板**（用 Commitizen）：

```
<type>(<scope>): <subject>

<body>

Refs: #<issue-id>
```

---

## CI / CD

```yaml
# GitHub Actions 必须通过的检查
- pnpm install --frozen-lockfile
- pnpm lint
- pnpm typecheck
- pnpm test --coverage
- pnpm build
- pnpm e2e:smoke
```

> **一人开发原则**：CI 必须 < 5 分钟。超过就要拆。

---

## 代码评审（自评清单）

每周五 E2E 后用这个清单自评：

- [ ] diff 是否 < 400 行？
- [ ] 没有顺手 refactor 不相关的代码？
- [ ] 没有引入新依赖（如必须，需在 PR 描述解释）？
- [ ] 单测覆盖新代码？
- [ ] E2E 覆盖新流程？
- [ ] 错误路径都有日志？
- [ ] Sentry breadcrumb 是否覆盖关键节点？
- [ ] 没有 PII 写到日志？
- [ ] 没有把 `.env` 提交？
- [ ] ADR 是否需要新增？

---

## 上下文管理（一人开发最容易踩的坑）

| 现象 | 解决 |
|---|---|
| 改 A 时想起来 B 也要改 | 记到 issue，不要当场改 |
| Cursor 上下文混乱 | 每个 issue 单独开 Cursor 会话 |
| Claude Code 跑长任务忘记目标 | 用 `task-list.md` 跟踪 |
| 跨天重启后失忆 | 用 `daily-standup` skill 生成昨日总结 |

---

## 工具替换决策矩阵

> 工具升级 / 替换前必须满足：

| 维度 | 阈值 |
|---|---|
| 效率提升 | ≥ 30% |
| 学习成本 | < 3 天 |
| 不增加月度成本 > ¥200 | （除非能省回来）|
| 不引入新依赖 | |