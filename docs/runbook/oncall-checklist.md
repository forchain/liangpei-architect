# Runbook · 每日 / 每周 / 上线前 自检清单

> 一人开发没有 peer review，但你必须**自我 review**。本清单是强制项。

---

## 每日自检（10 分钟 · 早 9:00）

### 数据面
- [ ] Sentry 错误率 < 0.5%（看 P95 / 错误数趋势）
- [ ] 数据库连接池使用率 < 70%
- [ ] Redis 内存使用 < 70%
- [ ] COS 桶大小无暴增
- [ ] DAU 数字符合预期（不大跌不大涨）

### 业务面
- [ ] 匹配池今日已生成（看 `match_candidates` 最新日期）
- [ ] 审核队列 pending < 50（人工介入阈值 500）
- [ ] 支付对账昨日 diff = 0
- [ ] 投诉 / 举报 < 5 条
- [ ] 推送到达率 > 90%

### AI 面
- [ ] DeepSeek-V3 成功率 > 99%
- [ ] AI 红娘首响 P95 < 3s
- [ ] LLM 兜底话术触发率 < 5%
- [ ] ASR / TTS 错误率 < 1%

### 安全 / 合规
- [ ] 紧急工单（priority=urgent）已全部处理
- [ ] 高优先级工单无超 30 分钟未处理
- [ ] 黑名单更新已生效

---

## 每周自检（30 分钟 · 周五 14:00）

### 跑全套 E2E
- [ ] 注册 → 登录 → 资料 → 匹配 → IM → 支付 → 退款 全链路通过
- [ ] AI 红娘对话 ≥ 5 轮流畅
- [ ] AI 分身问答给出合理回答
- [ ] AI 军师报告卡生成

### 数据库
- [ ] `pg_stat_statements` Top 10 慢查询已 review
- [ ] 索引缺失 / 膨胀已修复
- [ ] 备份恢复演练一次（随机挑一张表，从备份恢复）

### 代码
- [ ] `pnpm test --coverage` ≥ 80%
- [ ] `pnpm lint` 0 warning
- [ ] `pnpm typecheck` 0 error
- [ ] 依赖更新 `pnpm outdated` review

### 文档
- [ ] 本周新决策写 ADR
- [ ] Runbook 有更新（踩到的坑）
- [ ] README 关键数字对得上

### 业务指标
- [ ] WAU / MAU / 留存率（d1 / d7 / d30）
- [ ] 付费转化率
- [ ] AI 使用人均次数
- [ ] 客诉率（每千单）

---

## 上线前自检（每次发版前必跑）

### 功能
- [ ] 新功能在 staging 跑 ≥ 24h 无 P0 / P1
- [ ] 灰度 1% → 10% → 50% → 100% 各 2 小时无异常
- [ ] 取消订阅 / 退款流程 100% 通

### 性能
- [ ] Locust 1000 并发，P95 < 1.5s，错误率 < 0.5%
- [ ] 首屏 < 1.5s（小程序真机测）
- [ ] App 冷启动 < 3s

### 兼容
- [ ] iOS 16 / 17 实机测试
- [ ] Android 11 / 13 实机测试
- [ ] 微信开发者工具 + 真机各跑一遍
- [ ] 网络弱网（2G / 弱 Wi-Fi）测试

### 安全
- [ ] 关键接口鉴权 / 签名校验
- [ ] SQL 注入 / XSS 扫描（OWASP ZAP）
- [ ] 依赖漏洞 `pnpm audit`
- [ ] .env / 密钥未提交
- [ ] 用户 PII 未出现在日志

### 监控
- [ ] 新功能埋点已加 Sentry breadcrumb
- [ ] 关键指标已加 Prometheus alert
- [ ] 降级开关可用并验证

### 回滚
- [ ] 上一版本代码已 tag
- [ ] 数据库迁移可回滚（已写 DOWN）
- [ ] 5 分钟内可回滚到上一版本

### 沟通
- [ ] 已发种子用户灰度公告
- [ ] 客服微信已同步本次变更
- [ ] Runbook 已记录新功能的风险点

---

## 月度复盘（每月最后一个周日 · 1 小时）

### 业务
- DAU / MAU 趋势
- 付费转化
- 用户反馈 Top 5
- 竞品新动作

### 技术
- P0 / P1 故障复盘
- LLM 成本走势
- 性能退化点
- 工具升级评估

### 合规
- 监管政策变化（婚恋类目 / 支付 / AI）
- 资质到期提醒（ICP / 备案 / 商户号）
- 隐私政策更新

### 个人
- 倦怠指数（连续 3 周加班 → 强制休假）
- 学习计划（本月学了什么）
- 下月 OKR

---

## 紧急一键脚本（放在 `~/bin/`）

```bash
#!/bin/bash
# ~/bin/liangpei-health
# 一次性看核心指标

echo "=== Sentry 错误率（最近 1 小时）==="
curl -s "$SENTRY_API/api/0/projects/liangpei/api/stats/?stat=rate&resolution=1h" \
  -H "Authorization: Bearer $SENTRY_TOKEN" | jq .

echo "=== 数据库状态 ==="
ssh app-server-1 "pg_isready -h pg-primary -p 5432"

echo "=== AI 服务状态 ==="
curl -I https://api.liangpei.app/health/ai
curl -I https://api.liangpei.app/health/im

echo "=== 最近 10 条订单 ==="
psql -h pg-primary -d liangpei -c "
SELECT id, user_id, amount_cents, status, created_at
FROM orders ORDER BY created_at DESC LIMIT 10;"
```

---

## 一人开发的铁律

1. **永远不要相信"上线就稳了"**——上线只是开始
2. **永远不要在周五下午 5 点后发版**
3. **永远不要在睡梦中开着未监控的实验**
4. **永远不要等用户来报 bug**——主动监控
5. **永远不要一个人扛 P0**——必要时拉种子用户帮忙验证