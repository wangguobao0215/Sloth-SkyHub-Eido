# Playbook 引擎与 10 个核心剧本

> 本文档定义 Sloth-SkyHub-Eido 的 Playbook 引擎架构、标准 YAML 结构、命名规范，并详述 10 个覆盖客户全生命周期的核心 Playbook。

---

## Playbook 概念

**Playbook = 结构化、可自动化的客户成功流程**

```
触发条件 → 步骤序列 → 分支判断 → 升级机制 → 度量指标
```

Playbook 是 CSM 的标准操作手册，将最佳实践编码为可重复、可度量、可持续优化的流程。每个 Playbook 绑定明确的触发条件和成功指标，既可由 CSM 手动启动，也可由系统自动触发。

---

## 标准 YAML 结构

所有 Playbook 的 Obsidian frontmatter 遵循以下结构：

```yaml
playbook_id: "PB-HND-001"
name: "新客户标准Onboarding"
version: "1.0.0"
lifecycle_stage: "Handoff → Onboarding"
customer_tiers: ["Strategic", "Growth"]
owner: "CS Ops"
last_updated: "2026-04-23"
status: "active"                # active | draft | deprecated | archived
trigger:
  type: "event"                 # event | schedule | manual | health_score
  conditions:
    - "contract_signed = true"
    - "day_since_signing <= 1"
metrics:
  primary: "TTV"
  secondary: ["onboarding_completion_rate", "first_value_milestone_achieved"]
estimated_duration: "60d"
escalation_owner: "CS Manager"
```

---

## 命名规范

**格式：** `PB-{阶段代码}-{序号}-{中文名}.md`

| 阶段代码 | 含义 | 示例 |
|---------|------|------|
| HND | Handoff（移交） | PB-HND-001-新客户标准Onboarding.md |
| ONB | Onboarding（上线） | PB-ONB-001-首次价值里程碑冲刺.md |
| ADP | Adoption（采纳） | PB-ADP-001-低活跃用户激活.md |
| VAL | Value（价值验证） | PB-VAL-001-QBR全流程自动化.md |
| EXP | Expansion（扩展） | PB-EXP-001-扩展机会培育.md |
| RNW | Renewal（续约） | PB-RNW-001-续约倒计时120天.md |
| ALL | Cross-stage（跨阶段） | PB-ALL-001-客户健康分骤降响应.md |
| SCL | Scale（规模化） | PB-SCL-001-Scale客户自动生命周期.md |

---

## 10 个核心 Playbook

---

### 1. PB-HND-001 — 新客户标准 Onboarding

**触发条件：** `contract_signed = true AND day_since_signing <= 1`（Event 触发）
**适用层级：** Strategic, Growth
**预期时长：** 60 天

**步骤序列：**

1. **D0 — 内部移交会议：** 销售 → CSM 完成客户移交，同步合同条款、干系人地图、客户期望、历史沟通记录。生成客户笔记初始版。
2. **D1 — 欢迎邮件 + 启动会邀请：** 发送品牌化欢迎邮件，附启动会议程草案，确认参会人员及时间。
3. **D3-5 — 启动会（Kickoff）：** 双方团队介绍，确认 Success Plan 初稿（含业务 KPI、里程碑定义），明确沟通节奏。
4. **D7-14 — 技术部署 + 数据接入：** 协调技术团队完成环境部署、数据迁移、SSO 集成。每日跟踪阻塞项。
5. **D14-21 — 管理员培训：** 为客户管理员提供产品培训，交付培训材料和操作手册。
6. **D21-30 — 终端用户培训 + 初始推广：** 分批次培训终端用户，上线使用监控仪表盘。
7. **D30-45 — 首次价值检查点：** 检查核心功能使用率，与客户确认早期价值感知，调整 Success Plan。
8. **D60 — Onboarding 总结会：** 回顾里程碑达成情况，正式过渡到 Adoption 阶段，确认后续互动节奏。

**分支条件：**

- 若 D14 技术部署未完成 → 触发升级至 CS Manager + 客户技术负责人专项会议
- 若 D30 核心功能使用率 < 30% → 追加定制化培训 + 使用激励计划
- 若客户为 Scale 层级 → 自动转入 PB-SCL-001 简化流程

**升级规则：**

| 条件 | 升级对象 | 时限 |
|------|---------|------|
| 技术阻塞超过 7 天 | CS Manager + Product | 24h 内响应 |
| 关键干系人失联超 14 天 | CS Director | 48h 内响应 |
| D45 仍未完成启动会 | CS VP | 立即升级 |

**成功指标：**

| 指标 | 目标 |
|------|------|
| Onboarding 完成率 | >= 90% 在 60 天内完成 |
| TTV（首次价值实现时间） | <= 45 天 |
| 客户满意度（启动会后 CSAT） | >= 4.5/5 |

---

### 2. PB-ONB-001 — 首次价值里程碑冲刺

**触发条件：** `TTV > 60d OR milestone_deviation > 20%`（Health Score 触发）
**适用层级：** Strategic, Growth
**预期时长：** 30 天

**步骤序列：**

1. **诊断分析：** 拉取使用数据，定位价值实现的阻塞点（功能未启用 / 流程未嵌入 / 数据质量差）。
2. **干系人对齐会议：** 与客户 Champion 确认原始业务 KPI 是否仍有效，必要时调整里程碑定义。
3. **制定加速计划：** 输出 14 天冲刺计划，包含每日 / 每周具体动作项和责任人。
4. **密集辅导期（2 周）：** CSM 每周 2 次 check-in，提供 1-on-1 操作指导。
5. **Quick Win 展示：** 整理已取得的量化成果，向客户 Sponsor 汇报。
6. **里程碑确认 + 庆祝：** 正式确认首次价值里程碑达成，发送内部成功案例通报。

**分支条件：**

- 若阻塞点为产品缺陷 → 提交产品反馈 ticket，同步客户临时解决方案
- 若客户团队资源不足 → 建议客户引入内部 Champion，协调培训日程

**升级规则：** 冲刺计划执行 14 天后 TTV 仍无改善 → 升级至 CS Manager 介入。

**成功指标：** 首次价值里程碑达成率 >= 85%；冲刺后 30 天内 usage_depth 提升 >= 15 分。

---

### 3. PB-ADP-001 — 低活跃用户激活

**触发条件：** `DAU/MAU < 20% 持续 14 天`（Health Score 触发）
**适用层级：** All
**预期时长：** 30 天

**步骤序列：**

1. **使用数据下钻：** 按部门 / 角色拆解活跃度，定位低活跃人群。
2. **根因假设：** 与客户管理员沟通，判断原因（培训不足 / 流程未嵌入 / 替代工具 / 组织变动）。
3. **定制激活方案：** 根据根因设计干预措施（再培训 / 工作流模板 / 使用竞赛 / 管理层倡导）。
4. **执行激活计划：** 落地干预措施，每周追踪 DAU/MAU 变化。
5. **效果评估 + 固化：** 30 天后评估活跃度回升情况，固化有效做法。

**分支条件：**

- 若根因为替代工具竞争 → 输出竞品对比材料，安排高级功能演示
- 若根因为组织变动 → 转入 PB-ALL-002 关键人变动应急

**升级规则：** 30 天后 DAU/MAU 仍 < 25% → 升级至 CS Manager，评估是否为流失前兆。

**成功指标：** DAU/MAU 恢复至 >= 30%；激活用户 30 天留存率 >= 70%。

---

### 4. PB-VAL-001 — QBR 全流程自动化

**触发条件：** `季度日历触发（Q+1 第一个工作日前 21 天）`（Schedule 触发）
**适用层级：** Strategic, Growth
**预期时长：** 21 天（准备 14 天 + 执行 + 跟进 7 天）

**步骤序列：**

1. **D-21 — 数据自动汇总：** Skill 自动拉取健康分趋势、使用数据、工单统计、里程碑进度，生成 QBR 数据包。
2. **D-14 — CSM 叙事编排：** CSM 基于数据包编写价值叙事：回顾期成果 → 未达成项根因 → 下季度建议。
3. **D-10 — 内部预审：** CS Manager 审核 QBR 材料，确保叙事聚焦业务价值而非功能列表。
4. **D-7 — 客户确认议程：** 发送 QBR 议程和预读材料给客户，确认参会人名单。
5. **D0 — QBR 执行：** 结构化执行 QBR（60-90 分钟），记录客户反馈和 action items。
6. **D+1 — 会后总结发送：** 24 小时内发送 QBR 纪要，包含 action items 及负责人和时限。
7. **D+7 — Action Items 首次跟进：** 追踪 action items 进展，更新客户笔记。
8. **D+30 — Action Items 闭环：** 确认所有 action items 完成或转入下季度计划。

**分支条件：**

- 若客户为 Strategic 且 ARR > 阈值 → 升级为 EBR（Executive Business Review），邀请双方高管参加
- 若客户拒绝参会 → 触发 stakeholder_engagement 评分下调，CSM 尝试替代方案（书面报告 + 短会）

**升级规则：** 连续 2 个季度客户拒绝 QBR → 升级至 CS Director 介入关系修复。

**成功指标：** QBR 覆盖率 >= 90%（Strategic）/ >= 75%（Growth）；QBR 后 30 天内 action items 完成率 >= 80%。

---

### 5. PB-ALL-001 — 客户健康分骤降响应

**触发条件：** `health_score 7 天内下降 >= 15 分`（Health Score 触发）
**适用层级：** All
**预期时长：** 14 天紧急响应

**步骤序列：**

1. **T+0h — 自动告警：** Skill 检测到骤降后即时通知 CSM，附带七维度变化明细。
2. **T+4h — 根因定位：** CSM 分析哪些维度驱动了下降（使用骤降 / 工单激增 / 干系人沉默等）。
3. **T+24h — 紧急干预计划：** 根据根因制定 7 天干预方案，通知 CS Manager。
4. **T+48h — 客户沟通：** 主动联系客户关键联系人，了解背景、表达关注、确认协作方案。
5. **D3-7 — 密集执行：** 每日跟踪干预措施执行情况，持续监控健康分变化。
6. **D7 — 中期评估：** 评估健康分是否止跌，决定是否升级或调整策略。
7. **D14 — 闭环复盘：** 记录根因、干预措施、效果，沉淀为案例库。

**分支条件：**

- 若骤降由单一维度驱动（如 P0 工单） → 聚焦该维度专项修复
- 若多维度同时恶化 → 判定为系统性风险，直接升级

**升级规则：**

| 条件 | 升级对象 |
|------|---------|
| D7 健康分未止跌 | CS Manager |
| 健康分降至 Red（< 40） | CS Director |
| Strategic 客户健康分降至 Red | CS VP + Account Team |

**成功指标：** 健康分在 14 天内止跌企稳（降幅 < 5）；30 天内恢复至骤降前 80% 水平。

---

### 6. PB-EXP-001 — 扩展机会培育

**触发条件：** 系统检测到扩展信号（Health Score 触发 + Event 触发）
**扩展信号定义：**

- usage_depth >= 80 且 activity_trend >= 75
- 客户主动询问未购买功能
- 用户席位使用率 > 90%
- QBR 中客户提及新业务需求

**适用层级：** Strategic, Growth
**预期时长：** 60-90 天

**步骤序列：**

1. **信号确认 + 机会记录：** 验证扩展信号真实性，在客户笔记中创建扩展机会条目。
2. **需求挖掘：** 与客户 Champion 深入沟通，理解业务驱动力和预算周期。
3. **价值场景设计：** 基于现有使用数据，构建扩展产品的 ROI 预估。
4. **内部协同：** 与销售 AE 对齐扩展策略，明确角色分工（CSM 领导价值叙事，AE 领导商务谈判）。
5. **Sponsor 展示：** 向客户 Executive Sponsor 展示扩展方案和预期业务价值。
6. **推进商务流程：** 协助 AE 推进报价、审批、签约流程。
7. **扩展 Onboarding：** 签约后转入新模块 Onboarding（可复用 PB-HND-001 的简化版）。

**分支条件：**

- 若客户预算周期不匹配 → 标记为"长期培育"，降低跟进频率，设置下季度提醒
- 若客户对 ROI 存疑 → 安排 POC 或试用期

**升级规则：** 扩展机会金额 > 阈值 → 自动通知 Sales VP 和 CS VP 共同关注。

**成功指标：** 扩展机会转化率 >= 30%；扩展收入占 NRR 增量的 >= 50%。

---

### 7. PB-RNW-001 — 续约倒计时 120 天

**触发条件：** `合同到期前 120 天自动触发`（Schedule 触发）
**适用层级：** All
**预期时长：** 120 天

**步骤序列：**

1. **D-120 — 续约评估启动：** 自动生成续约评估报告：健康分、使用趋势、价值实现、干系人关系、商务历史。
2. **D-110 — 内部对齐：** CSM + AE + CS Manager 三方对齐，评估续约风险等级和策略。
3. **D-90 — 客户续约意向沟通：** CSM 与客户 Champion 沟通续约意向，收集初步反馈。
4. **D-75 — 价值回顾材料准备：** 汇总合同期内客户取得的业务成果，编写价值总结。
5. **D-60 — 正式续约提案：** AE 发出续约提案（含价格、条款、新增建议），CSM 配合价值叙事。
6. **D-30 — 谈判 + 审批推进：** 跟踪客户内部审批流程，解答疑虑，处理条款协商。
7. **D-14 — 签约冲刺：** 密集跟进，确保在到期前完成签约。
8. **D0 — 续约完成 / 流失记录：** 完成续约或记录流失根因分析。

**分支条件：**

- 若 D-90 客户表达不续约意向 → 立即转入 PB-RNW-002 高风险客户挽留
- 若客户要求缩减合同规模 → 启动价值重申 + 分析缩减影响
- 若健康分为 Red → 并行启动 PB-ALL-001 健康分骤降响应

**升级规则：**

| 条件 | 升级对象 |
|------|---------|
| D-60 仍无续约意向反馈 | CS Manager |
| D-30 仍未签约且健康分 Yellow/Red | CS Director |
| Strategic 客户 D-14 仍未签约 | CS VP |

**成功指标：** GRR >= 90%；续约提前完成率（D-30 前签约） >= 60%。

---

### 8. PB-RNW-002 — 高风险客户挽留

**触发条件：** `续约风险 >= Level 2`（Health Score 触发 + Manual 触发）
**风险等级定义：**

- Level 1：健康分 Yellow + 客户未表态 → PB-RNW-001 覆盖
- Level 2：健康分 Red 或客户明确表达犹豫 → 触发本 Playbook
- Level 3：客户正式通知不续约 → 本 Playbook + 管理层直接介入

**适用层级：** Strategic, Growth
**预期时长：** 30-60 天

**步骤序列：**

1. **T+0 — 紧急内部战情会：** CSM + CS Manager + AE + 产品代表，分析流失根因，制定挽留策略。
2. **T+2d — Executive 介入：** CS Director/VP 致信客户 Executive Sponsor，表达重视。
3. **T+5d — 深度倾听会：** 与客户决策层 1-on-1 沟通，不推销，只倾听痛点和不满。
4. **T+7d — 挽留方案制定：** 基于倾听反馈，制定针对性挽留方案（产品改进承诺 / 服务升级 / 价格调整 / 专属资源）。
5. **T+10d — 挽留方案提案：** 向客户正式提交挽留方案，附带时间承诺和验收标准。
6. **T+14-30d — 执行 + 验证：** 兑现承诺，密集跟进，持续向客户展示改善。
7. **T+30d — 续约决策推动：** 基于改善成果，推动客户做出续约决策。
8. **结案记录：** 无论成败，详细记录根因、挽留措施、客户反馈、结果，沉淀到案例库。

**分支条件：**

- 若根因为产品能力不足 → 提交至 Product Board，承诺路线图时间线
- 若根因为价格 → 由 AE 主导商务谈判，CSM 提供价值论据支持
- 若客户已签约竞品 → 记录竞品信息，执行优雅退出（保持关系以待未来回签）

**升级规则：** Level 3 风险立即升级至 CS VP + CRO 层级。

**成功指标：** 挽留成功率 >= 40%（Level 2）/ >= 20%（Level 3）；挽留客户 12 个月留存率 >= 70%。

---

### 9. PB-ALL-002 — 关键人变动应急

**触发条件：** Champion 或 Executive Sponsor 离职 / 转岗（Event 触发）
**适用层级：** Strategic, Growth
**预期时长：** 30 天

**步骤序列：**

1. **T+0 — 信息确认：** 确认变动信息来源和可靠性，更新干系人地图。
2. **T+1d — 影响评估：** 评估离开者的影响力等级和接替者情况，更新健康分。
3. **T+2d — 继任者接触：** 第一时间联系继任者或临时负责人，自我介绍，表达合作意愿。
4. **T+5d — 关系重建会议：** 与继任者举行介绍会，回顾合作历史、当前价值、进行中的项目。
5. **T+7d — 新 Champion 培育：** 识别并开始培育新的内部 Champion（如果继任者不适合该角色）。
6. **T+14d — 稳定性评估：** 评估新关系的稳定性，确认合作节奏。
7. **T+30d — 关系正常化确认：** 确认新干系人关系达到正常互动水平。

**分支条件：**

- 若离开者为唯一 Champion → 紧急启动多线程关系建设（同时培育 2-3 个潜在 Champion）
- 若继任者对产品持负面态度 → 并行启动价值重申 + 管理层关系加固
- 若组织发生大规模变动 → 升级为组织变动专项应对

**升级规则：** Executive Sponsor 变动 → 立即通知 CS Director；30 天内未建立新 ES 关系 → 升级至 CS VP。

**成功指标：** 新 Champion 建立时间 <= 21 天；关键人变动后 90 天内健康分波动 <= 10 分。

---

### 10. PB-SCL-001 — Scale 客户自动生命周期

**触发条件：** `customer_tier = "Scale"`（Event 触发：客户分类）
**适用层级：** Scale（专用）
**预期时长：** 持续运行

**步骤序列：**

1. **自动 Onboarding：** 触发自助 Onboarding 邮件序列（Day 1/3/7/14/30），附带视频教程和帮助中心链接。
2. **使用监控 + 自动干预：** 系统监控使用数据，低活跃自动发送再激活邮件；高活跃自动发送高级功能推荐。
3. **自动健康评估：** 月度自动计算健康分，Red 客户自动转入人工 CSM 池。
4. **自助 QBR 报告：** 每季度自动生成使用报告邮件，客户可自助查看。
5. **续约自动化：** 到期前 90 天自动发送续约提醒邮件；到期前 30 天未响应 → 升级至人工。
6. **NPS 自动收集：** 季度自动发送 NPS 调查，Detractor 自动转入人工池。

**分支条件：**

- 若 Scale 客户健康分持续 Green 且使用增长 → 评估是否升级至 Growth 层级
- 若 Scale 客户健康分降至 Red → 自动分配 CSM，转入标准 Playbook 流程
- 若 Scale 客户主动联系请求人工支持 → 临时分配 CSM，评估是否永久升级层级

**升级规则：** 任何 Scale 客户 ARR 增长超过 Growth 层级阈值 → 自动提醒 CS Ops 评估层级升级。

**成功指标：** Scale 客户 GRR >= 85%；人工介入率 <= 15%；自动化邮件打开率 >= 40%。

---

## Obsidian 存储结构

```
02-Playbooks/
├── lifecycle/                  # 按生命周期阶段存放模板
│   ├── PB-HND-001-新客户标准Onboarding.md
│   ├── PB-ONB-001-首次价值里程碑冲刺.md
│   ├── PB-ADP-001-低活跃用户激活.md
│   ├── PB-VAL-001-QBR全流程自动化.md
│   ├── PB-EXP-001-扩展机会培育.md
│   ├── PB-RNW-001-续约倒计时120天.md
│   └── PB-RNW-002-高风险客户挽留.md
├── cross-stage/                # 跨阶段通用 Playbook
│   ├── PB-ALL-001-客户健康分骤降响应.md
│   └── PB-ALL-002-关键人变动应急.md
├── scale/                      # Scale 客户专用
│   └── PB-SCL-001-Scale客户自动生命周期.md
├── executions/                 # Playbook 执行记录
│   ├── {客户名}/
│   │   ├── PB-HND-001_2026-Q2_执行记录.md
│   │   └── PB-ALL-001_2026-04-20_骤降响应.md
│   └── ...
└── _templates/                 # Playbook 执行模板
    ├── execution_log_template.md
    └── escalation_template.md
```

---

## Playbook 执行记录 Frontmatter 模板

```yaml
execution_id: "EX-PB-HND-001-Acme-2026Q2"
playbook_id: "PB-HND-001"
customer: "Acme Corp"
started: 2026-04-01
status: "in_progress"         # in_progress | completed | aborted | escalated
current_step: 4
escalation_count: 0
owner_csm: "Alice Chen"
notes: ""
completed_date:
outcome:                      # success | partial | failed
```

---

## Dataview 查询：活跃 Playbook 看板

```dataview
TABLE
  playbook_id AS "Playbook",
  customer AS "客户",
  current_step AS "当前步骤",
  status AS "状态",
  owner_csm AS "CSM",
  started AS "启动日期"
FROM "02-Playbooks/executions"
WHERE status = "in_progress"
SORT started ASC
```

## Dataview 查询：升级中的 Playbook

```dataview
TABLE
  playbook_id AS "Playbook",
  customer AS "客户",
  escalation_count AS "升级次数",
  owner_csm AS "CSM"
FROM "02-Playbooks/executions"
WHERE status = "escalated"
SORT escalation_count DESC
```

---

## Playbook并发与冲突管理

### 优先级规则

当同一客户同时触发多个 Playbook 时，按以下优先级执行：

| 优先级 | Playbook | 说明 |
|--------|----------|------|
| P0 | PB-RNW-002 高风险挽留 | 最高优先级，所有其他 Playbook 暂停 |
| P1 | PB-ALL-001 健康分骤降 | 紧急响应优先 |
| P1 | PB-ALL-002 关键人变动 | 紧急响应优先 |
| P2 | PB-RNW-001 续约倒计时 | 商业优先级 |
| P3 | PB-EXP-001 扩展培育 | 增长型，可被 P0-P2 暂停 |
| P4 | PB-VAL-001 QBR 自动化 | 常规运营 |
| P4 | PB-HND-001 标准 Onboarding | 常规运营 |
| P4 | PB-SCL-001 Scale 自动化 | 常规运营 |
| P5 | PB-ONB-001 TTV 加速 | 可延迟 |
| P5 | PB-ADP-001 低活跃激活 | 可延迟 |

### 并发处理规则

1. **同优先级可并行**：如 PB-VAL-001 和 PB-HND-001 可同时执行
2. **高优先级暂停低优先级**：P0 触发时，P3-P5 自动暂停；P0 完成后，暂停的 Playbook 自动恢复
3. **互斥 Playbook**：PB-EXP-001（扩展）与 PB-RNW-002（挽留）互斥——不能同时追增购又做挽留
4. **最大并发数**：同一客户最多 3 个活跃 Playbook，超出时按优先级排队

### 过期与清理规则

| 状态 | 条件 | 系统行为 |
|------|------|---------|
| `stale` | `in_progress` 超过预期时长 1.5 倍且无步骤更新 | 自动标记，通知 CSM |
| `stale` 持续 7 天 | CSM 未响应 | 推送确认：继续 / 暂停 / 关闭 |
| 关闭 | CSM 选择关闭 | 必须选择 outcome：`completed` / `partial` / `aborted_customer_unresponsive` / `aborted_superseded` / `aborted_other` |

### 结果归因

每个 Playbook 执行记录必须包含 `outcome_attribution` 字段：

```yaml
outcome_attribution:
  primary_metric: "health_score"
  before_value: 48
  after_value: 72
  attribution_confidence: "medium"
  notes: "健康分提升主要由关键干系人重新建联驱动"
```
