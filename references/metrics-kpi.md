# 度量体系与 KPI

> 本文档定义 Sloth-SkyHub-Eido 的三层金字塔指标架构，覆盖商业指标、过程指标、效率指标，明确指标间的驱动链关系，并提供 Obsidian Dataview 仪表盘查询。

---

## 三层金字塔指标架构

```
        ┌─────────────────┐
        │   L0 商业指标    │  季度/年度 — 结果层
        │  NRR · GRR · ER │
        └────────┬────────┘
                 │ 驱动
        ┌────────┴────────┐
        │   L1 过程指标    │  月度 — 过程层
        │ TTV · HS · QBR  │
        └────────┬────────┘
                 │ 驱动
        ┌────────┴────────┐
        │   L2 效率指标    │  周度 — 效率层
        │ CSM效率 · AI采纳 │
        └─────────────────┘
```

---

## L0 商业指标（季度/年度）

公司级客户成功结果指标，直接与收入挂钩。

### NRR — 净收入留存率 (Net Revenue Retention)

**公式：**

```
NRR = (期初ARR + 扩展收入 - 缩减收入 - 流失收入) / 期初ARR * 100%
```

| 项目 | 说明 |
|------|------|
| 目标 | >= 110%（年度） |
| 数据源 | CRM（Salesforce）合同记录 + 财务系统 |
| 计算频率 | 季度滚动 + 年度终值 |
| 归属 | CS VP + CRO 共同负责 |

**因果关系：** NRR 由扩展收入驱动上行，由缩减和流失驱动下行。CSM 通过提升采纳率和发现扩展机会推高 NRR。

---

### GRR — 总收入留存率 (Gross Revenue Retention)

**公式：**

```
GRR = (期初ARR - 缩减收入 - 流失收入) / 期初ARR * 100%
```

| 项目 | 说明 |
|------|------|
| 目标 | >= 90%（年度） |
| 数据源 | CRM 合同记录 + 财务系统 |
| 计算频率 | 季度滚动 + 年度终值 |
| 归属 | CS VP 主要负责 |

**因果关系：** GRR 反映客户留存的底线能力。健康分 Green 占比越高、续约 Playbook 执行越充分，GRR 越稳。

---

### Expansion Revenue Rate — 扩展收入率

**公式：**

```
Expansion Revenue Rate = 期间扩展收入 / 期初ARR * 100%
```

| 项目 | 说明 |
|------|------|
| 目标 | >= 20%（年度） |
| 数据源 | CRM 扩展合同 + Upsell/Cross-sell 记录 |
| 计算频率 | 季度 |
| 归属 | CS + Sales 共同负责 |

**因果关系：** 扩展机会来源于产品深度使用（usage_depth >= 80）+ 价值实现（value_realization 里程碑达成）。PB-EXP-001 是核心驱动 Playbook。

---

### Logo Churn Rate — 客户流失率

**公式：**

```
Logo Churn Rate = 期间流失客户数 / 期初客户总数 * 100%
```

| 项目 | 说明 |
|------|------|
| 目标 | <= 5%（年度） |
| 数据源 | CRM 合同到期未续约记录 |
| 计算频率 | 季度 |
| 归属 | CS VP 主要负责 |

**因果关系：** Logo Churn 是 GRR 的客户数版本。健康分 Red 客户是流失高风险群体，PB-RNW-002 挽留 Playbook 直接影响此指标。

---

## L1 过程指标（月度）

CS 团队运营层面的过程质量指标。

### TTV — 首次价值实现时间 (Time to Value)

**定义：** 从合同签署到客户确认达成首个业务价值里程碑的天数。

| 项目 | 说明 |
|------|------|
| 目标 | Strategic <= 45 天；Growth <= 30 天；Scale <= 14 天（自助） |
| 数据源 | Obsidian 客户笔记 `first_value_date - contract_signed_date` |
| 计算频率 | 月度 |
| 归属 | CSM 个人 + CS Ops |

**因果关系：** TTV 越短，客户早期留存率越高。PB-HND-001 和 PB-ONB-001 直接驱动 TTV 缩短。

---

### Health Score Distribution — 健康分分布

**定义：** 客户按健康等级（Green/Yellow/Red）的分布比例。

| 项目 | 说明 |
|------|------|
| 目标 | Green >= 65%；Yellow <= 25%；Red <= 10% |
| 数据源 | Obsidian 客户笔记 `health_level` 字段 |
| 计算频率 | 月度 |
| 归属 | CS VP + CS Ops |

**因果关系：** Green 占比越高，GRR 和 NRR 越稳。健康分骤降响应（PB-ALL-001）控制 Red 占比。

---

### Playbook Completion Rate — Playbook 完成率

**定义：** 已完成 Playbook 执行数 / 已触发 Playbook 执行数 * 100%

| 项目 | 说明 |
|------|------|
| 目标 | >= 85% |
| 数据源 | Obsidian `02-Playbooks/executions/` 记录 |
| 计算频率 | 月度 |
| 归属 | CS Ops |

**因果关系：** Playbook 完成率直接反映 CSM 执行力。低完成率意味着客户风险未被系统化处理。

---

### QBR Coverage — QBR 覆盖率

**定义：** 实际执行 QBR 的客户数 / 应执行 QBR 的客户数 * 100%

| 项目 | 说明 |
|------|------|
| 目标 | Strategic >= 95%；Growth >= 80% |
| 数据源 | Obsidian QBR 执行记录 |
| 计算频率 | 季度 |
| 归属 | CSM 个人 |

---

### EBR Execution Rate — EBR 执行率

**定义：** 实际执行 EBR 的 Strategic 客户数 / Strategic 客户总数 * 100%

| 项目 | 说明 |
|------|------|
| 目标 | >= 80%（年度至少 1 次） |
| 数据源 | Obsidian EBR 执行记录 |
| 计算频率 | 年度 |
| 归属 | CS Director + CSM |

---

## L2 效率指标（周度）

CSM 个人和工具效率指标，驱动 L1 过程质量。

### CSM Efficiency — CSM 效率

| 子指标 | 定义 | 目标 |
|--------|------|------|
| 客户数/人 | CSM 负责的客户数量 | Strategic: 10-15; Growth: 25-40; Scale: 200+（自动化） |
| ARR/人 | CSM 负责的 ARR 总额 | 依公司规模而定，逐年基准化 |
| 有效工时占比 | 客户互动 + 策略性工作时间 / 总工时 | >= 70% |

---

### AI Adoption Rate — AI 采纳率

**定义：** CSM 使用 Skill 辅助完成工作的频率。

```
AI Adoption Rate = 使用 Skill 完成的任务数 / 总可 Skill 辅助任务数 * 100%
```

| 项目 | 说明 |
|------|------|
| 目标 | >= 60%（3 个月内）；>= 80%（6 个月内） |
| 数据源 | Skill 调用日志 |
| 计算频率 | 周度 |
| 归属 | CS Ops |

---

### Skill Call Frequency — Skill 调用频次

**定义：** 每位 CSM 每周调用 Skill 的次数。

| 项目 | 说明 |
|------|------|
| 目标 | >= 20 次/周/人 |
| 数据源 | Skill 调用日志 |
| 计算频率 | 周度 |
| 归属 | CS Ops |

---

### Response Time — 客户响应时间

**定义：** 从客户提出请求到 CSM 首次实质性响应的平均时长。

| 项目 | 说明 |
|------|------|
| 目标 | Strategic <= 2h；Growth <= 4h；Scale <= 24h（自动回复除外） |
| 数据源 | CRM 活动记录 + 邮件/IM 时间戳 |
| 计算频率 | 周度 |
| 归属 | CSM 个人 |

---

### Skill Success Rate — Skill 成功率

**定义：** Skill 调用后输出被 CSM 采纳（未做重大修改直接使用）的比例。

| 项目 | 说明 |
|------|------|
| 目标 | >= 75% |
| 数据源 | Skill 调用日志 + CSM 反馈标记 |
| 计算频率 | 月度 |
| 归属 | CS Ops（用于 Skill 优化） |

---

## 驱动链：L2 → L1 → L0

指标之间的因果驱动关系，用于诊断问题根源和预测趋势。

### 链路 1：Skill 调用 → Playbook 完成 → NRR

```
Skill调用频次 ↑ → Playbook完成率 ↑ → 客户健康分 Green占比 ↑
                                      → 扩展机会发现率 ↑ → NRR ↑
```

**逻辑：** CSM 更频繁使用 Skill 辅助执行 Playbook，每个 Playbook 的执行质量和及时性更高，客户健康度改善，扩展机会更多被捕获。

### 链路 2：AI 采纳 → 响应时间 → 健康分 → GRR

```
AI采纳率 ↑ → 响应时间 ↓ → stakeholder_engagement ↑
                          → support_tickets 改善
                          → 健康分 Green占比 ↑ → GRR ↑
```

**逻辑：** AI 辅助减少 CSM 重复劳动，释放时间用于客户互动，改善干系人关系和工单处理效率，推高整体健康分。

### 链路 3：TTV → 首年扩展 → NRR

```
TTV ↓ → 客户早期满意度 ↑ → 首年续约率 ↑
                          → 首年扩展收入 ↑ → NRR ↑
```

**逻辑：** 客户越早感受到价值，越早进入采纳深化阶段，首年内产生扩展需求的概率越高。

### 链路 4：QBR 覆盖 → 价值认知 → GRR

```
QBR覆盖率 ↑ → 客户价值认知 ↑ → value_realization 分数 ↑
                                → 续约意愿 ↑ → GRR ↑
```

**逻辑：** 定期的 QBR 帮助客户看到产品带来的业务价值，强化续约意愿。

---

## CSM 仪表盘 Dataview 查询

### 风险预警看板 — Red 客户一览

```dataview
TABLE
  health_score AS "健康分",
  health_30d_delta AS "30d变化",
  lifecycle_stage AS "阶段",
  tier AS "层级",
  csm AS "CSM",
  renewal_date AS "续约日期",
  dateformat(renewal_date - date(today), "d '天'") AS "距续约"
FROM "01-Customers"
WHERE health_level = "Red"
SORT health_score ASC
```

### 快速恶化监控 — 30 天下降 > 10 分

```dataview
TABLE
  health_score AS "当前分",
  health_30d_delta AS "30d变化",
  health_level AS "等级",
  csm AS "CSM",
  tier AS "层级"
FROM "01-Customers"
WHERE health_30d_delta < -10
SORT health_30d_delta ASC
```

### CSM 个人看板 — 健康分布 + 活跃 Playbook + 续约日历

**健康分布：**

```dataview
TABLE
  length(filter(rows, (r) => r.health_level = "Green")) AS "Green",
  length(filter(rows, (r) => r.health_level = "Yellow")) AS "Yellow",
  length(filter(rows, (r) => r.health_level = "Red")) AS "Red",
  length(rows) AS "总数"
FROM "01-Customers"
WHERE csm = "{{当前CSM}}"
GROUP BY true
```

**活跃 Playbook：**

```dataview
TABLE
  playbook_id AS "Playbook",
  customer AS "客户",
  current_step AS "当前步骤",
  started AS "启动日期"
FROM "02-Playbooks/executions"
WHERE owner_csm = "{{当前CSM}}" AND status = "in_progress"
SORT started ASC
```

**续约日历（未来 120 天）：**

```dataview
TABLE
  health_score AS "健康分",
  health_level AS "等级",
  tier AS "层级",
  renewal_date AS "续约日期",
  dateformat(renewal_date - date(today), "d '天'") AS "距续约"
FROM "01-Customers"
WHERE csm = "{{当前CSM}}"
  AND renewal_date >= date(today)
  AND renewal_date <= date(today) + dur(120 days)
SORT renewal_date ASC
```

### 管理层看板 — 团队 NRR + 风险热力图 + Skill 使用

**团队 NRR 概览：**

```dataview
TABLE
  length(rows) AS "客户数",
  sum(rows.arr) AS "总ARR",
  sum(rows.expansion_revenue) AS "扩展收入",
  round(sum(rows.expansion_revenue) / sum(rows.arr) * 100, 1) + "%" AS "扩展率",
  length(filter(rows, (r) => r.health_level = "Red")) AS "Red数"
FROM "01-Customers"
GROUP BY csm AS "CSM"
SORT sum(rows.arr) DESC
```

**风险热力图（按层级 x 健康等级）：**

```dataview
TABLE
  length(filter(rows, (r) => r.health_level = "Green")) AS "Green",
  length(filter(rows, (r) => r.health_level = "Yellow")) AS "Yellow",
  length(filter(rows, (r) => r.health_level = "Red")) AS "Red"
FROM "01-Customers"
GROUP BY tier AS "客户层级"
```

**Skill 使用统计：**

```dataview
TABLE
  skill_calls_this_week AS "本周调用",
  skill_calls_last_week AS "上周调用",
  round((skill_calls_this_week - skill_calls_last_week) / skill_calls_last_week * 100, 1) + "%" AS "环比",
  skill_success_rate AS "成功率"
FROM "04-Team/CSM-Profiles"
SORT skill_calls_this_week DESC
```

---

## 指标治理

### 数据刷新频率

| 层级 | 刷新频率 | 延迟容忍度 |
|------|---------|-----------|
| L0 | 季度 | T+5 工作日 |
| L1 | 月度 | T+2 工作日 |
| L2 | 周度 | T+1 工作日 |

### 指标 Owner 矩阵

| 指标 | 定义 Owner | 数据 Owner | 消费者 |
|------|-----------|-----------|--------|
| NRR | Finance | CS Ops | CS VP, CRO, Board |
| GRR | Finance | CS Ops | CS VP, Board |
| TTV | CS Ops | CSM | CS Manager, CSM |
| Health Score | CS Ops | 系统自动 + CSM | CSM, CS Manager |
| Playbook Completion | CS Ops | CSM | CS Manager |
| CSM Efficiency | CS Ops | HR + CS Ops | CS VP |
| AI Adoption | CS Ops | 系统日志 | CS Ops, CS VP |

### 异常处理规则

- 数据缺失：采用最近一次有效值，标记 `data_stale: true`
- 异常值：偏离均值 3 个标准差以上自动标记，人工审核后决定是否纳入计算
- 新客户：合同签署后前 30 天不纳入团队级 L0/L1 统计（避免新客户稀释指标）

---

## VoC反馈闭环流程

> 定义 VoC（客户之声）从数据采集到闭环验证的 6 步完整流程，包含采集渠道、情感分析引擎、分级响应路由（P0-P3）、升级处理机制和客户情感信号分类策略。

### 完整闭环流程总览

```
┌─────────────────────────────────────────────────────────────────────┐
│                    VoC 反馈闭环全流程                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ① 数据采集        ② 情感分析        ③ 分级响应                     │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐                    │
│  │ NPS调研   │────▶│ AI情感    │────▶│ 自动分级  │                    │
│  │ CSAT问卷  │     │ 分类引擎  │     │ 路由引擎  │                    │
│  │ 企微反馈  │     │          │     │          │                    │
│  │ 工单评价  │     └──────────┘     └────┬─────┘                    │
│  │ 会议纪要  │                          │                           │
│  └──────────┘                          ▼                           │
│                              ┌─────────────────┐                   │
│                              │ Promoter(9-10)   │──▶ 转介绍激活     │
│                              │ Passive(7-8)     │──▶ 提升计划       │
│                              │ Detractor(0-6)   │──▶ 紧急升级       │
│                              └────────┬────────┘                   │
│                                       │                            │
│  ⑥ 闭环验证        ⑤ 产品反馈        ④ 升级处理                    │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐                    │
│  │ 客户确认  │◀────│ 产品团队  │◀────│ CS Leader │                    │
│  │ 满意度复测│     │ 需求评审  │     │ 48h介入   │                    │
│  │ 闭环存档  │     │ 排期反馈  │     │ 根因分析  │                    │
│  └──────────┘     └──────────┘     └──────────┘                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 环节一：数据采集

| 采集渠道 | 采集方式 | 频率 | 存储位置 |
|----------|----------|------|----------|
| NPS调研 | 产品内弹窗 / 邮件问卷 | 季度 | `VoC/NPS/` 目录 |
| CSAT | 工单关闭后自动触发 | 事件驱动 | `VoC/CSAT/` 目录 |
| 企微/钉钉反馈 | CSM手动标记 或 AI识别负面关键词 | 实时 | 客户笔记内 |
| 工单评价 | 工单关闭后用户评分 | 事件驱动 | 工单系统 |
| 会议纪要 | QBR/EBR中的客户反馈段落（AI自动提取） | 按会议 | 客户笔记内 |

### 环节二：情感分析（AI Skill 执行）

VoC Analyzer Skill 对采集到的文本进行分类：

```
输入：客户原始反馈文本
处理：
  1. 情感极性判断：正面 / 中性 / 负面
  2. 情感强度评分：1-5（5为最强烈）
  3. 主题分类（见下方分类表）
  4. 是否涉及竞品提及
  5. 是否包含流失信号关键词
输出：结构化情感分析报告（JSON格式 → 写入客户笔记）
```

### 环节三：分级响应路由（P0-P3）

| 反馈类型 | 响应等级 | 响应时效 | 响应负责人 | 触发动作 |
|----------|----------|----------|------------|----------|
| Detractor（NPS 0-6） | P0 | 24小时内 | CS Manager + CSM | 创建紧急Playbook，通知管理层 |
| 负面情感（强度 >= 4） | P1 | 48小时内 | CSM + 必要时升级 | 触发"客户挽救"Playbook |
| 竞品提及 | P1 | 48小时内 | CSM + 竞对雷达 Skill | 生成竞对应对方案 |
| Passive（NPS 7-8） | P2 | 7天内 | CSM | 制定提升计划 |
| 负面情感（强度 < 4） | P2 | 7天内 | CSM | 加入下次触达议程 |
| Promoter（NPS 9-10） | P3 | 14天内 | CSM | 触发"转介绍/案例合作"Playbook |

### 环节四至六：升级处理 -> 产品反馈 -> 闭环验证

- **升级处理**：CS Manager 在 48 小时内完成根因分析，区分"产品问题"、"服务问题"、"预期管理问题"
- **产品反馈**：若为产品问题，通过标准化模板提交至产品需求池，包含客户影响面、紧急度、建议解决方案
- **产品回复**：产品团队在 5 个工作日内回复处理方案（接受/排期/拒绝+理由）
- **闭环验证**：CSM 将处理结果同步客户，确认满意度。若客户不满意，重新进入升级流程

### 客户情感信号分类与响应策略

| 信号类别 | 典型表现 | 信号强度 | 响应策略 |
|----------|----------|----------|----------|
| **流失预警** | "在看其他方案"、"领导问有没有替代品"、停止参加QBR | 极高 | 立即升级 CS Director，24h内启动高层对话 |
| **价值质疑** | "用了半年感觉没效果"、"ROI算不出来" | 高 | 触发 Value Realization Skill，48h内准备价值证明报告 |
| **功能不满** | "XX功能太难用"、"竞品有但你们没有" | 中高 | 记录产品反馈，72h内给客户解决时间表 |
| **服务不满** | "响应太慢"、"换了三个CSM了" | 中高 | CSM主管亲自跟进，制定服务改进承诺 |
| **预期落差** | "跟销售说的不一样"、"当初承诺的功能呢" | 高 | 回溯销售承诺，协调产品/Sales对齐预期 |
| **组织变动** | "我们部门要调整"、"负责人要换了" | 中 | 启动"干系人变更"Playbook，快速建立新关系 |
| **积极信号** | "用起来很顺手"、"帮我们省了很多时间" | 正面 | 趁热打铁，推动案例合作/转介绍 |
| **扩展意向** | "其他部门也想用"、"能不能加购模块" | 正面高 | 立即通知 AM/Sales，48h内准备扩展方案 |

---

## CSM负载指数 (CSM Workload Index)

### 定义

加权客户数，衡量 CSM 当前工作负载。

### 计算公式

```
CSM Workload Index = (Strategic客户数 × 3) + (Growth客户数 × 1) + (Scale客户数 × 0.1)
                   + (活跃Playbook数 × 0.5)
                   + (未来90天到期续约数 × 2)
```

| 项目 | 说明 |
|------|------|
| 目标 | 团队中位数的 0.7-1.3 倍 |
| 预警 | > 1.5 倍中位数触发 CS Manager 评估 |
| 数据源 | Obsidian 客户笔记 + Playbook 执行记录 |
| 计算频率 | 周度 |
| 归属 | CS Ops |

### Dataview查询

```dataview
TABLE
  length(filter(rows, (r) => r.tier = "Strategic")) * 3 +
  length(filter(rows, (r) => r.tier = "Growth")) * 1 +
  length(filter(rows, (r) => r.tier = "Scale")) * 0.1 AS "负载指数",
  length(rows) AS "客户总数"
FROM "01-Customers"
GROUP BY csm
SORT "负载指数" DESC
```

### 再平衡触发规则

1. 指数 > 1.5 倍团队中位数 → 自动通知 CS Manager
2. 指数 > 2.0 倍中位数 → 强制再平衡（转移部分 Growth 客户）
3. 未来 90 天内同一 CSM 有 >= 5 个续约到期 → 预警并建议分流
