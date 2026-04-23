# 健康评分模型详解

> 本文档定义 Sloth-SkyHub-Eido 的客户健康评分（Health Score）七维度模型，包括各维度计算逻辑、阶段动态权重、等级阈值、季度校准流程，以及 Obsidian 集成方式。

---

## 七维度模型

| 维度 | 英文ID | 默认权重 | 范围 | 说明 |
|------|--------|---------|------|------|
| 产品使用深度 | usage_depth | 25% | 0-100 | 核心功能覆盖率 + 高级功能渗透率 |
| 活跃度趋势 | activity_trend | 20% | 0-100 | 30天 DAU/MAU 及环比变化 |
| 商务关系 | commercial_health | 15% | 0-100 | 回款状态 + 合同剩余天数 + 历史续约行为 |
| 支持工单 | support_tickets | 10% | 0-100 | 工单量趋势 + 未解决 P0/P1 数 + 平均解决时长 |
| 干系人关系 | stakeholder_engagement | 15% | 0-100 | 关键干系人互动频率 + Executive Sponsor 活跃度 |
| 价值实现 | value_realization | 10% | 0-100 | 客户定义的业务 KPI 达成率 |
| NPS/满意度 | sentiment | 5% | 0-100 | 最新 NPS 评分 + CSAT 均值 |

**综合健康分** = SUM(各维度得分 * 对应权重)

---

## 各维度计算逻辑

### 1. usage_depth — 产品使用深度

**公式：**

```
usage_depth = core_coverage * 0.6 + advanced_penetration * 0.4
```

- `core_coverage` = 已使用核心功能数 / 合同包含核心功能总数 * 100
- `advanced_penetration` = 已使用高级功能数 / 可用高级功能总数 * 100

**分段映射：**

| 原始分 | 映射分 |
|--------|--------|
| >= 80 | 100 |
| 60 - 79 | 75 |
| 40 - 59 | 50 |
| 20 - 39 | 25 |
| < 20 | 10 |

**数据源：** 产品分析平台（Mixpanel / Amplitude / 内部埋点）每日同步。

---

### 2. activity_trend — 活跃度趋势

**基础分计算：**

```
基础分 = min(DAU / MAU * 100 * 1.5, 100)
```

**趋势修正：**

| 环比变化 | 修正值 |
|---------|--------|
| 增长 > 20% | +10 |
| 增长 5% - 20% | +5 |
| -5% ~ +5% | 0 |
| 下降 -5% ~ -20% | -10 |
| 下降 < -20% | -20 |

**最终分 = clamp(基础分 + 趋势修正, 0, 100)**

**数据源：** 产品后台 DAU/MAU 接口，按自然月计算环比。

---

### 3. commercial_health — 商务关系

**三因子加权：**

```
commercial_health = 回款因子 * 0.4 + 合同到期因子 * 0.3 + 续约历史因子 * 0.3
```

**回款因子：**

| 状态 | 得分 |
|------|------|
| 无逾期 | 100 |
| 逾期 < 30 天 | 70 |
| 逾期 30 - 90 天 | 40 |
| 逾期 > 90 天 | 10 |

**合同到期因子：**

| 剩余天数 | 得分 |
|---------|------|
| > 180 天 | 100 |
| 90 - 180 天 | 80 |
| 30 - 90 天 | 50 |
| < 30 天 | 20 |

**续约历史因子：**

| 历史行为 | 得分 |
|---------|------|
| 连续续约 2 年+ | 100 |
| 续约 1 次 | 70 |
| 首次合同 | 50 |
| 曾缩减合同规模 | 30 |

**数据源：** CRM（Salesforce / HubSpot）+ 财务系统回款记录。

---

### 4. support_tickets — 支持工单

**起始分：100，逐项扣分。**

| 条件 | 扣分 |
|------|------|
| 每个未解决 P0 | -25 |
| 每个未解决 P1 | -15 |
| 每个未解决 P2 | -5 |
| 工单量激增（环比 > 50%） | -10 |
| SLA 违规（至少 1 次） | -10 |
| 零工单（过去 30 天） | +5（奖励） |

**最终分 = max(累计分, 0)**

**数据源：** 工单系统（Zendesk / Jira Service Desk / Intercom）。

---

### 5. stakeholder_engagement — 干系人关系

**双因子加权：**

```
stakeholder_engagement = 关键干系人互动 * 0.6 + Executive Sponsor 活跃度 * 0.4
```

**关键干系人互动频率：**

| 频率 | 得分 |
|------|------|
| 周互动 | 100 |
| 双周互动 | 80 |
| 月度互动 | 60 |
| 季度互动 | 30 |
| 无互动 | 0 |

**Executive Sponsor（ES）活跃度：**

| 活跃度 | 得分 |
|--------|------|
| 月度活跃 | 100 |
| 季度活跃 | 60 |
| 半年活跃 | 30 |
| 无 ES 或完全沉默 | 20 |

**数据源：** CRM 活动记录 + 日历系统会议记录 + CSM 手动更新。

---

### 6. value_realization — 价值实现

**基础分：**

```
value_realization = 已达成里程碑数 / 计划里程碑数 * 100
```

**特殊修正规则：**

- **Onboarding 期间（合同签署后前 90 天）：** 自动给予 60 分宽限期基础分，避免新客户因尚未完成部署而被误判为低价值实现。
- **CSM 手动覆盖：** 支持 CSM 手动设置分数（需在 `value_override_reason` 字段注明理由），供校准会议审核。

**数据源：** Success Plan 里程碑追踪（Obsidian 笔记 / 客户成功平台）。

---

### 7. sentiment — NPS/满意度

**NPS 映射：**

| NPS 评分 | 得分 |
|---------|------|
| 9 - 10（Promoter） | 100 |
| 7 - 8（Passive） | 60 |
| <= 6（Detractor） | 20 |

**回退策略：**

1. 优先使用最新一次 NPS 评分
2. 若无 NPS，使用近 90 天 CSAT 均值（1-5分制，线性映射到 0-100）
3. 若无任何调研数据，默认 50 分（中性），并自动触发 **"需收集反馈"** 提醒任务

**数据源：** 调研工具（Delighted / SurveyMonkey / Typeform）。

---

## 阶段动态权重

客户在不同生命周期阶段关注重点不同，权重随阶段自动调整：

| 维度 | Onboarding (0-90d) | Adoption (91-365d) | 稳态 (2年+) | 续约窗口 (最后90d) |
|------|---------------------|---------------------|-------------|---------------------|
| usage_depth | 20% | 30% | 25% | 20% |
| activity_trend | 30% | 20% | 20% | 15% |
| commercial_health | 5% | 10% | 15% | 25% |
| support_tickets | 15% | 10% | 10% | 10% |
| stakeholder_engagement | 10% | 15% | 15% | 20% |
| value_realization | 15% | 10% | 10% | 5% |
| sentiment | 5% | 5% | 5% | 5% |

**阶段判定逻辑：**

- 续约窗口优先级最高：合同到期前 90 天自动切入续约窗口权重
- Onboarding：合同签署后 0-90 天
- Adoption：91-365 天
- 稳态：合同执行超过 365 天且不在续约窗口内

---

## 健康等级阈值

| 等级 | 分数区间 | 含义 | 推荐动作 |
|------|---------|------|---------|
| **Green** | >= 70 | 健康 | 维持节奏，寻找扩展机会 |
| **Yellow** | 40 - 69 | 需关注 | 启动诊断 Playbook，加密互动频次 |
| **Red** | < 40 | 高风险 | 立即启动挽留 Playbook，升级至管理层 |

---

## 季度校准流程

每季度末执行一次模型校准，确保评分准确性持续提升。

### 步骤

1. **数据回测** — 拉取上季度所有流失客户，验证流失前 30 天健康分分布；统计上季度评分准确率。
2. **CSM 反馈修正** — 收集 CSM 标记的"分数偏高"和"分数偏低"案例，形成偏差清单。
3. **校准会议** — CS VP + CS Ops + Team Leads 参会，审核偏差案例，决策是否调整维度权重或阈值。
4. **版本管理** — 更新后的权重方案存储在 `_config/health_score_versions/`，文件名格式 `hs_v{版本号}_{日期}.yaml`。

### 校准目标

| 指标 | 目标值 |
|------|-------|
| 预测准确率（分数 < 40 且实际流失） | >= 60% |
| 漏报率（实际流失但分数 >= 70） | <= 10% |

---

## Obsidian Frontmatter 字段模板

每个客户笔记的 frontmatter 应包含以下健康评分字段：

```yaml
health_score: 72
health_level: "Green"
health_model_version: "v1.0"
health_updated: 2026-04-23
health_dimensions:
  usage_depth: 80
  activity_trend: 75
  commercial_health: 65
  support_tickets: 85
  stakeholder_engagement: 60
  value_realization: 70
  sentiment: 50
health_trend: "stable"         # stable | improving | declining
health_30d_delta: -3
health_override: false
health_override_reason: ""
```

---

## Dataview 查询示例

### 风险客户预警列表

```dataview
TABLE
  health_score AS "健康分",
  health_level AS "等级",
  health_30d_delta AS "30d变化",
  lifecycle_stage AS "阶段",
  csm AS "CSM",
  renewal_date AS "续约日期"
FROM "01-Customers"
WHERE health_level = "Red"
SORT health_score ASC
```

### 快速恶化监控（30 天下降 > 10 分）

```dataview
TABLE
  health_score AS "当前分",
  health_30d_delta AS "30d变化",
  health_level AS "等级",
  csm AS "CSM"
FROM "01-Customers"
WHERE health_30d_delta < -10
SORT health_30d_delta ASC
```

### 健康分分布统计

```dataview
TABLE
  length(filter(rows, (r) => r.health_level = "Green")) AS "Green",
  length(filter(rows, (r) => r.health_level = "Yellow")) AS "Yellow",
  length(filter(rows, (r) => r.health_level = "Red")) AS "Red"
FROM "01-Customers"
GROUP BY csm AS "CSM"
```

### 需收集反馈提醒

```dataview
LIST
FROM "01-Customers"
WHERE health_dimensions.sentiment = 50 AND health_override = false
SORT health_score ASC
```
