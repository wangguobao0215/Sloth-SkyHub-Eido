# 跨 Skill 协同与数据流

> 定义 SkyHub 15 个 Skill 之间的依赖关系、数据流转规则、编排模式与 Sloth 家族集成方案。

---

## Skill 依赖图

```
Sales-Handoff --> TTV-Tracker --> Adoption-Analyzer --> Value-Proof-Generator --> Report-Builder
                                        |                      |
Health-Dashboard <-- Voice-of-Customer   |                      |
     |                                   v                      v
Churn-Predict --> Renewal-War-Room <-- Competitor-Radar    Executive-Briefing
     |                |
Empathy-Writer   Workflow-Agent

KB-QA (通用基座，所有 Skill 均可调用)
Log-Analyzer (独立运行，技术支持场景)
Role-Play-Coach (独立运行，培训场景)
```

### 依赖类型说明

| 符号 | 含义 | 示例 |
|------|------|------|
| `-->` | 数据单向流动 | Sales-Handoff 输出交接文档供 TTV-Tracker 消费 |
| `<--` | 反向数据注入 | Voice-of-Customer 的情感数据注入 Health-Dashboard |
| `v` | 触发式调用 | Adoption-Analyzer 分析结果触发 Value-Proof-Generator |

---

## 四条核心数据链

### Chain 1: 客户生命周期链

**路径:** Sales-Handoff -> TTV-Tracker -> Adoption-Analyzer -> Value-Proof-Generator -> Report-Builder

**用途:** 追踪客户从签约到价值实现的完整旅程，自动积累价值证据。

| 环节 | 触发条件 | 数据格式 | 示例 |
|------|----------|----------|------|
| Sales-Handoff -> TTV-Tracker | 交接文档完成并标记 `status: handed-off` | YAML frontmatter: `expected_ttv`, `key_milestones[]`, `stakeholders[]` | 客户完成签约，交接文档包含 90 天 Onboarding 计划 |
| TTV-Tracker -> Adoption-Analyzer | 首次价值实现里程碑达成或超时 | JSON: `{ttv_days, milestones_completed[], blockers[]}` | TTV 达成后开始追踪深度采纳指标 |
| Adoption-Analyzer -> Value-Proof-Generator | 采纳阶段评估完成（月度/季度） | JSON: `{adoption_score, feature_usage{}, trend, recommendations[]}` | 采纳分 >= 70 自动触发价值报告生成 |
| Value-Proof-Generator -> Report-Builder | 价值证明数据就绪 | Markdown + YAML: ROI 计算、使用数据、客户证言 | 续约前 60 天自动生成价值回顾报告 |

---

### Chain 2: 风险响应链

**路径:** Health-Dashboard -> Churn-Predict -> Renewal-War-Room -> Empathy-Writer

**用途:** 从风险识别到干预行动的快速响应闭环。

| 环节 | 触发条件 | 数据格式 | 示例 |
|------|----------|----------|------|
| Health-Dashboard -> Churn-Predict | 健康分降至黄色或红色区间 | JSON: `{customer_id, health_score, level, dimensions{}, drop_reason}` | 客户健康分从 75 降至 55，触发流失预测 |
| Churn-Predict -> Renewal-War-Room | 流失概率 >= 40% | JSON: `{churn_probability, risk_factors[], recommended_actions[]}` | 预测 60% 流失概率，启动续约作战室 |
| Renewal-War-Room -> Empathy-Writer | 作战室确定干预方案后需沟通 | JSON: `{action_type, tone, key_points[], customer_context{}}` | 生成挽留邮件：强调已交付价值 + 新功能路线图 |

---

### Chain 3: 升级链

**路径:** Voice-of-Customer -> Health-Dashboard -> Executive-Briefing

**用途:** 严重客户情感信号自动升级至管理层。

| 环节 | 触发条件 | 数据格式 | 示例 |
|------|----------|----------|------|
| Voice-of-Customer -> Health-Dashboard | 检测到 Detractor（NPS <= 6）或严重负面情感 | JSON: `{sentiment_score, category, verbatim, source, urgency}` | NPS 回复 3 分，备注 "考虑更换供应商" |
| Health-Dashboard -> Executive-Briefing | 健康分降至红色且客户 ARR >= 阈值 | JSON: `{customer_summary, risk_timeline, financial_impact, action_needed}` | 年 ARR 50 万的客户连续 2 周红色，自动生成高管简报 |

---

### Chain 4: 续约防御链

**路径:** Expansion-Radar + Competitor-Radar -> Renewal-War-Room

**用途:** 续约窗口期内整合扩展机会与竞品威胁，制定攻防策略。

| 环节 | 触发条件 | 数据格式 | 示例 |
|------|----------|----------|------|
| Expansion-Radar -> Renewal-War-Room | 续约前 90 天且存在扩展机会 | JSON: `{expansion_signals[], estimated_uplift, recommended_bundle}` | 检测到 3 个部门有新需求，预估扩展 30% |
| Competitor-Radar -> Renewal-War-Room | 续约前 90 天且检测到竞品活动 | JSON: `{competitor, activity_type, evidence[], defense_talking_points[]}` | 竞品在客户公司做了 POC 演示 |
| 合并 -> Renewal-War-Room | 上述任一或两者同时触发 | 综合分析：机会 + 威胁 -> 续约策略 | 作战室整合数据，输出 "攻守兼备" 续约方案 |

---

## 上下文传递规则

### 会话上下文 (Session Context)

- 在同一对话轮次中，Skill 之间通过 MCP 会话共享上下文
- 包含：当前客户 ID、用户角色、对话历史摘要
- 生命周期：对话结束即释放

### 持久上下文 (Persistent Context)

- 通过 Obsidian frontmatter 更新实现跨会话持久化
- 每次 Skill 执行后将关键输出写入客户笔记的 frontmatter
- 字段规范：`last_skill`, `last_action`, `last_updated`, `pending_followup`

### 事件驱动 (Event-driven)

- Skill 输出满足特定条件时自动触发 Playbook
- 触发器定义在 Playbook 引擎中，Skill 本身不感知下游
- 示例：Health-Dashboard 输出 `level: red` -> 触发 "风险干预" Playbook

---

## 编排模式

### 1. 顺序执行 (Sequential)

```
A --> B --> C
```

- 前一个 Skill 完成后，输出作为下一个 Skill 的输入
- 示例：Sales-Handoff -> TTV-Tracker -> Adoption-Analyzer

### 2. 并行执行 (Parallel)

```
A + B --> C
```

- 多个 Skill 同时执行，结果汇聚后传入下游
- 示例：Expansion-Radar + Competitor-Radar -> Renewal-War-Room

### 3. 条件分支 (Conditional)

```
IF condition THEN A ELSE B
```

- 根据运行时数据选择不同 Skill 路径
- 示例：IF health_level == "red" THEN Churn-Predict ELSE Adoption-Analyzer

### 4. 扇出 (Fan-out)

```
A --> B, C, D (simultaneously)
```

- 一个 Skill 的输出同时分发给多个下游 Skill
- 示例：Voice-of-Customer --> Health-Dashboard, Executive-Briefing, Empathy-Writer

---

## 数据格式标准

### 通用字段规范

| 字段 | 类型 | 说明 |
|------|------|------|
| `customer_id` | string | 全局唯一客户标识，与 CRM 保持一致 |
| `timestamp` | ISO 8601 | 数据产生时间 |
| `source_skill` | string | 产生该数据的 Skill 名称 |
| `confidence` | float (0-1) | 数据或预测的置信度 |

### 健康数据格式

```yaml
health:
  score: 72
  level: yellow  # green(>=75) / yellow(50-74) / red(<50)
  dimensions:
    adoption: 80
    engagement: 65
    support: 70
    outcome: 75
    sentiment: 60
  updated_at: "2026-04-23T10:00:00Z"
```

### 报告格式

- 正文：Markdown
- 元数据：YAML frontmatter
- 必含字段：`title`, `customer_id`, `generated_by`, `generated_at`, `report_type`

### 告警格式

```yaml
alert:
  priority: P1  # P0(立即) / P1(4h) / P2(24h) / P3(周内)
  source_skill: Churn-Predict
  customer_id: "CUST-001"
  message: "流失概率升至 65%，建议启动续约作战室"
  recommended_action: "trigger_renewal_war_room"
  created_at: "2026-04-23T10:00:00Z"
```

---

## Sloth 家族技能集成

SkyHub 与 Sloth-Role-Eido 家族中的其他技能无缝协作，形成端到端的客户成功生态。

| Sloth 家族 Skill | 集成 SkyHub Skill | 协同场景 |
|------------------|-------------------|----------|
| **Sloth-Sales-Eido** | Sales-Handoff | Sales-Eido 的 Pipeline 数据作为交接输入；BANT/MEDDIC 评估结果写入交接文档 |
| **Sloth-PSC-Eido** | Sales-Handoff, Competitor-Radar | PSC 会议纪要自动流入交接文档；竞品分析 Battle Card 同步至 Competitor-Radar |
| **Sloth-DeckBuilder-Eido** | Report-Builder | Report-Builder 生成的数据报告委托 DeckBuilder 自动转化为 PPT 演示文稿 |
| **Sloth-CPQ-Eido** | Renewal-War-Room | 续约定价、扩展报价由 CPQ 引擎计算，结果注入作战室方案 |
| **Sloth-StratAlign-Eido** | Value-Proof-Generator | 战略对齐分析结果作为 Value Realization 的高层级论据 |
| **Sloth-MGO-Eido** | Executive-Briefing | 管理层报告数据汇入高管简报，实现跨职能视图 |

### 集成数据流示例

```
[签约] Sloth-Sales-Eido (Pipeline)
         |
         v
[交接] Sloth-PSC-Eido (会议纪要) + Sales-Handoff
         |
         v
[Onboarding] TTV-Tracker --> Adoption-Analyzer
         |
         v
[价值证明] Value-Proof-Generator + Sloth-StratAlign-Eido
         |
         v
[续约] Sloth-CPQ-Eido (报价) + Renewal-War-Room
         |
         v
[汇报] Report-Builder --> Sloth-DeckBuilder-Eido (PPT)
                     --> Executive-Briefing + Sloth-MGO-Eido
```

---

## 集成注意事项

1. **松耦合原则:** Skill 之间通过标准数据格式通信，不直接调用内部方法
2. **幂等性:** 同一输入多次调用 Skill 应产生相同结果
3. **降级策略:** 上游 Skill 不可用时，下游 Skill 应能以有限数据运行或提示手动输入
4. **版本兼容:** 数据格式变更需向后兼容，新增字段设为 optional
5. **审计日志:** 所有跨 Skill 数据流转记录可追溯，存储在 `_meta/integration-log`

---

## 端到端场景演示

> 三个完整场景展示"深信 · 客户成功"系统从信号感知到闭环复盘的全流程运作方式，每个场景包含具体的输入输出示例。

---

### 场景一：国企大客户续费期危机处理

**背景设定**：
- 客户：某国有银行（Strategic 层级，ARR ￥300万）
- 现状：距续费到期还有 65 天，健康分从 72 降至 48（黄色预警）
- 触发信号：活跃度连续3周下降 + CSM在企微收到对方项目经理消息"领导在让我们调研其他产品"

---

**第一步：信号感知与风险识别**

```
触发源：
  1. 健康评分引擎检测到 health_30d_delta = -24（超过阈值-10）
  2. 企微消息中 VoC Analyzer Skill 识别到关键词"调研其他产品"（流失预警信号）
  
系统动作：
  - 自动将客户 health_level 从 Yellow 降级为 Red
  - 在客户笔记 risk_flags 中添加 ["竞品调研信号", "续费窗口期", "活跃度连续下降"]
  - 触发"续费危机"Playbook
  
通知输出（推送至CSM企微侧边栏）：
  ┌─────────────────────────────────────────┐
  │ ⚠️ 紧急预警：XX银行续费风险升级            │
  │                                          │
  │ 健康分：48 (↓24) | 层级：Red              │
  │ 续费到期：65天后                           │
  │ 风险信号：                                 │
  │   • 检测到竞品调研意向                      │
  │   • DAU连续3周环比下降（-15%/-12%/-18%）    │
  │   • 关键联系人互动频次降至月均1次            │
  │                                          │
  │ [查看详情] [启动续费保卫Playbook]           │
  └─────────────────────────────────────────┘
```

**第二步：父智能体意图解析与任务路由**

```
CSM输入（企微侧边栏）：
  "对方项目经理说领导在看别的产品，我该怎么办？快续费了，这个客户不能丢。"

父智能体（天枢中枢）解析：
  - 意图分类：[续费危机处理]（复合意图）
  - 需要调用的子技能：
    1. 客户体检报告 Skill → 全面体检，找到根因
    2. 竞对雷达 Skill → 分析可能的竞品及应对策略
    3. 流失预警 Skill → 评估流失概率
  - 执行方式：并行调用（三个Skill无依赖关系）
```

**第三步：多Skill并行执行**

**Skill A：客户体检报告**

```
输入：customer_id = "C-20240601-BANK"
执行过程：
  - 拉取近90天产品使用数据
  - 分析7个健康维度得分变化
  
输出：
  {
    "诊断摘要": "核心问题集中在产品使用深度和干系人关系两个维度",
    "关键发现": [
      "高级报表功能上线后使用率仅12%（行业均值45%）— 表明培训不足",
      "原对接副行长（Executive Sponsor）3个月前调岗，新负责人尚未建立关系",
      "近60天仅有2次有效互动（正常应为8次+）",
      "基础功能使用稳定，核心价值仍在"
    ],
    "根因判断": "不是产品本身问题，是高级功能未充分交付 + 关键干系人变动导致价值感知下降"
  }
```

**Skill B：竞对雷达**

```
输入：customer_id = "C-20240601-BANK", industry = "金融", signal = "调研其他产品"
执行过程：
  - 检索金融行业竞品情报库
  - 匹配该客户规模和需求特征
  
输出：
  {
    "可能竞品": ["竞品A（价格优势）", "竞品B（金融行业定制）"],
    "我方优势": ["数据安全合规（等保三级）", "已有6个月数据积累", "迁移成本高"],
    "我方劣势": ["高级功能交付不足", "近期服务响应感知差"],
    "建议话术要点": [
      "强调数据资产价值：半年数据不可迁移，重建成本约3个月",
      "强调合规优势：我方已通过金融行业等保三级，竞品需重新评估",
      "正面回应：承认高级功能培训不足，提出专项赋能计划"
    ]
  }
```

**Skill C：流失预警**

```
输入：customer_id = "C-20240601-BANK"
执行过程：
  - 运行流失概率评估规则
  - 匹配历史相似案例
  
输出：
  {
    "流失概率": "68%（高风险）",
    "关键风险因子": ["Executive Sponsor变更", "竞品调研", "活跃度持续下降"],
    "相似历史案例": {
      "案例名": "XX保险（2024年Q2流失）",
      "相似度": "85%",
      "当时失败原因": "发现竞品调研信号后未及时高层介入，拖延2周后客户已完成POC",
      "教训": "必须在72小时内完成高层对话"
    },
    "挽回建议": "立即安排VP级别对话，展示专项价值报告，提供续费激励方案"
  }
```

**第四步：父智能体编排与策略生成**

```
父智能体综合三个Skill结果，生成分阶段行动计划：

  ┌─────────────────────────────────────────────────┐
  │  XX银行续费保卫战 — 行动计划                       │
  ├─────────────────────────────────────────────────┤
  │                                                  │
  │  第一阶段：紧急止血（0-72小时）                     │
  │  ├─ 行动1：安排CS VP与对方新负责人电话（目标：建立   │
  │  │  信任，表达重视）                               │
  │  ├─ 行动2：CSM发送关怀消息（调用话术Skill生成）      │
  │  └─ 行动3：内部拉齐（Sales + Product + CS三方对齐） │
  │                                                  │
  │  第二阶段：价值重塑（第1-2周）                      │
  │  ├─ 行动4：安排高级功能专项培训（2次，覆盖报表模块） │
  │  ├─ 行动5：准备《XX银行专属价值报告》               │
  │  │  （调用 Value Realization Skill 生成）           │
  │  └─ 行动6：提供30天免费高级功能体验                 │
  │                                                  │
  │  第三阶段：锁定续费（第3-4周）                      │
  │  ├─ 行动7：安排正式QBR（展示价值报告 + 未来规划）    │
  │  ├─ 行动8：提出续费方案（含早续优惠）               │
  │  └─ 行动9：推动签约                               │
  │                                                  │
  └─────────────────────────────────────────────────┘
```

**第五步：话术生成与交付**

```
调用"企微高情商话术 Skill"，输入上下文后生成：

  版本A（关怀版 — 发给对方项目经理）：
  "王经理您好，最近了解到贵行在做新一轮的系统规划，我们非常理解也很重视。
   坦率说，过去两个月我们在高级功能的培训陪跑上做得不够到位，这是我们的责任。
   我已经跟公司申请了专项支持方案，想跟您约个时间当面聊聊，也听听您和新领导
   对系统的期望和建议。您看本周哪天方便？"

  版本B（高层版 — 发给对方新负责人，由CS VP发送）：
  "刘行长好，我是XX公司客户成功副总裁。贵行是我们最重要的金融行业客户之一，
   近期我们注意到在服务保障上有做得不够的地方，我想亲自向您汇报我们的改进计划
   和为贵行定制的专项方案。冒昧请您给我15分钟时间，我可以配合您的日程安排。"

  [复制版本A] [复制版本B] [修改后发送]
```

**第六步：执行跟踪与闭环**

```
Playbook引擎自动追踪：
  - 行动1（VP对话）：已完成，3月18日通话30分钟，对方态度缓和
  - 行动2（关怀消息）：已发送，对方回复"谢谢重视"
  - 行动3（内部对齐）：已完成
  - 行动4（专项培训）：进行中，第一次已完成，第二次安排在3月25日
  - 行动5（价值报告）：生成中
  - ...

健康分变化追踪：
  3月15日：48 -> 3月22日：52(+4) -> 3月29日：61(+9)
  
  状态：从 Red 回升至 Yellow，趋势 declining -> improving
```

**第七步：复盘存档**

```
续费结果：客户于4月底完成续费，合同金额维持￥300万，另签约高级模块￥50万

复盘总结（AI自动生成 + CSM审核后存入知识库）：
  - 时间线：从预警到续费签约共45天
  - 关键成功因素：VP高层介入及时（72h内）、价值报告打动新负责人、培训弥补了功能交付短板
  - 改进点：应在Executive Sponsor变更时立即触发关系重建Playbook，而非等到活跃度下降后
  - 沉淀为新规则：当检测到"关键干系人离职/调岗"时，自动将health_30d_delta扣减系数提高50%
```

---

### 场景二：新客户Onboarding全流程

**背景设定**：
- 客户：某互联网中型企业（Growth 层级，ARR ￥80万）
- 场景：Sales 签约完成，进入 CS 交接流程

---

**阶段一：Sales Handoff（Day 0-3）**

```
触发：Sales在CRM中将商机状态改为"Closed Won"

Sales Handoff Skill 自动执行：
  
  输入（从CRM + Sales笔记中提取）：
    - 客户基本信息（公司名、行业、规模）
    - 合同信息（产品版本、金额、合同期限、特殊条款）
    - 销售过程记录（客户需求、决策链、竞品对比、关键承诺）
    - 干系人信息（决策者、使用者、技术对接人）
  
  输出（自动生成Obsidian客户笔记）：
    1. 客户主笔记：含完整 YAML frontmatter
       - lifecycle_stage: "Onboarding"
       - health_score: 60（新客户初始值）
    2. Handoff摘要卡片：
       ┌─────────────────────────────────────┐
       │  新客户交接卡：XX互联网                │
       │  ARR: ￥80万 | 合同期: 1年             │
       │  购买模块：基础版 + 数据分析模块        │
       │  决策者：CTO 李总                      │
       │  日常对接：产品总监 陈经理              │
       │  关键承诺：                            │
       │    • Sales承诺30天内完成基础部署         │
       │    • 赠送2次高级培训                    │
       │  竞品背景：曾用竞品A，因XX原因更换       │
       │  客户核心诉求：提升团队协作效率30%       │
       │  注意事项：CTO技术背景深，重视API能力    │
       └─────────────────────────────────────┘
    3. 自动创建 Onboarding Playbook 实例
    4. 发送企微通知给指定CSM
```

**阶段二：启动会准备与执行（Day 3-7）**

```
Onboarding Playbook 步骤1：准备启动会（Kick-off Meeting）

CSM输入："帮我准备XX互联网的启动会材料"

系统调用链：
  1. 知识问答 Skill：检索该客户行业的Onboarding最佳实践
  2. 汇报文档生成 Skill：生成启动会PPT

PPT内容自动生成：
  - Slide 1：欢迎页 + 双方参会人介绍
  - Slide 2：项目目标与成功标准（基于Sales Handoff中的客户核心诉求）
  - Slide 3：Onboarding时间线与里程碑（90天计划）
  - Slide 4：双方职责矩阵（RACI）
  - Slide 5：沟通机制（周会 vs 月会、紧急联系方式）
  - Slide 6：下一步行动（技术对接安排）

CSM审核修改后 -> 发送给客户
```

**阶段三：技术部署与配置（Day 7-21）**

```
Onboarding Playbook 步骤2-4：技术部署

Playbook自动跟踪：
  步骤2：环境准备（客户侧）— 负责人：客户技术对接人
    - 自动发送环境准备checklist给客户
    - Day 10 未完成 -> 自动提醒CSM跟进
  
  步骤3：产品配置 — 负责人：实施团队
    - 配置完成后自动通知CSM
    - 故障排查助手 Skill 待命，处理配置问题
  
  步骤4：数据迁移（如有）— 负责人：实施团队
    - 迁移完成后自动触发数据校验

状态追踪（每日更新至客户笔记）：
  onboarding_progress:
    environment_ready: true     # Day 9 完成
    product_configured: true    # Day 14 完成
    data_migrated: true         # Day 18 完成
    admin_trained: false        # 进行中
```

**阶段四：用户培训与激活（Day 21-45）**

```
Onboarding Playbook 步骤5-6：培训与激活

  步骤5：管理员培训
    - 陪练教练 Skill 生成培训计划和quiz
    - 培训完成后自动发送考核问卷
    
  步骤6：终端用户激活
    - 监控 DAU/WAU 指标
    - IF Day 30 活跃用户数 < 目标值的 50% -> 自动告警
      CSM收到提醒："XX互联网用户激活率偏低（当前35%，目标70%），建议安排追加培训"
    - 调用话术 Skill 生成催促邮件模板，发给客户管理员
```

**阶段五：首个Value Milestone达成（Day 45-90）**

```
Onboarding Playbook 步骤7：价值验证

  Value Realization Skill 执行：
    输入：客户核心诉求 = "提升团队协作效率30%"
    
    数据采集：
      - 上线前基线数据（Sales阶段记录 或 客户提供）
      - 当前数据（从产品后台采集）
    
    输出（价值验证报告）：
      ┌──────────────────────────────────────────┐
      │  XX互联网 — 首月价值报告                    │
      │                                           │
      │  核心指标：团队协作效率                      │
      │  基线值：周均跨部门协作 12 次                │
      │  当前值：周均跨部门协作 19 次                │
      │  提升幅度：+58% 超出目标                    │
      │                                           │
      │  附加价值：                                 │
      │  • 审批流程平均耗时从 3天 降至 0.5天         │
      │  • 文档协作冲突率降低 72%                    │
      │                                           │
      │  建议：已达成首个价值里程碑，                 │
      │  建议推进高级功能采用（数据分析模块）          │
      └──────────────────────────────────────────┘

  TTV计算：
    合同签署日: Day 0
    首个Value Milestone达成日: Day 52
    TTV = 52天（目标 <= 60天）

  状态更新：
    lifecycle_stage: "Onboarding" -> "Adoption"
    health_score: 60 -> 75（激活成功加分）
    onboarding_completed: true
```

---

### 场景三：Tech-touch客户的全自动化服务

**背景设定**：
- 客户群体：200+ 个 Tech-touch 层级客户（ARR < ￥10万/户）
- CSM配比：1 个 CSM 管理全部 200+ 客户，无法逐一人工跟进
- 目标：通过全自动化流程保障基本服务质量，人工仅在必要时介入

---

**全自动化服务架构**

```
┌──────────────────────────────────────────────────────────────┐
│                 Tech-touch 全自动化服务链                      │
│                                                              │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐  │
│  │ 健康监测  │───▶│ 预警分级  │───▶│ 自动干预  │───▶│ 效果验证  │  │
│  │ (每日)   │    │ (实时)   │    │ (自动)   │    │ (周度)   │  │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘  │
│       │              │              │              │         │
│       ▼              ▼              ▼              ▼         │
│  自动采集数据     三级预警体系     分级自动响应     反馈闭环     │
│  计算健康分       判断干预级别     执行对应动作     验证效果     │
│                                                              │
│  ════════════════════════════════════════════════════════     │
│  人工介入触发条件：                                            │
│  • 健康分 < 30（极高风险）                                     │
│  • 自动干预后14天健康分未回升                                   │
│  • 客户主动要求人工对接                                        │
│  • 续费到期前30天仍为Red                                      │
│  ════════════════════════════════════════════════════════     │
└──────────────────────────────────────────────────────────────┘
```

**环节一：自动健康监测（每日执行）**

```
定时任务（每日 08:00 自动运行）：

  FOR EACH customer IN tech_touch_pool:
    1. 通过 MCP Server 采集产品使用数据
    2. 运行健康评分规则引擎（简化版，仅计算4个核心维度）：
       - 产品使用深度（30%）
       - 活跃度趋势（30%）
       - 支持工单（20%）
       - 商务关系（20%）
    3. 更新客户笔记 frontmatter
    4. 计算 health_30d_delta
    
  输出：200+ 客户的健康分日报（存入 Dashboards/tech_touch_daily.md）
```

**环节二：三级预警分级（实时触发）**

```
预警级别定义：

  Level 1 — 轻度异常（自动处理，不通知CSM）：
    触发条件：health_score 在 50-69 且 trending downward
    示例：某客户本周登录次数比上周少30%

  Level 2 — 中度风险（自动处理 + 通知CSM）：
    触发条件：health_score 在 30-49 或 health_30d_delta < -15
    示例：某客户两周未登录 + 有未解决工单

  Level 3 — 严重风险（通知CSM，建议人工介入）：
    触发条件：health_score < 30 或 连续3次自动干预无效
    示例：某客户1个月未登录 + 合同3个月后到期
```

**环节三：分级自动干预**

```
Level 1 自动干预（全自动，L4级）：

  触发后自动执行：
    1. 发送个性化邮件（模板 + AI填充客户信息）：
       主题："[客户名] 近期使用小贴士"
       内容：根据客户使用数据推荐未使用的功能 + 教程链接
    
    2. 产品内弹窗引导：
       推送与客户行业相关的最佳实践视频
    
    3. 14天后复查：
       IF 活跃度回升 -> 标记"干预成功"，关闭工单
       IF 未回升 -> 升级为 Level 2

---

Level 2 自动干预（自动 + CSM知晓，L3级）：

  触发后自动执行：
    1. 发送CSM署名的关怀邮件（AI生成，自动发送）：
       主题："[CSM名] 关心您的使用体验"
       内容：询问是否遇到困难 + 提供自助资源 + 预约1v1选项
    
    2. 创建轻量级待办（推送至CSM看板）：
       "XX客户14天活跃度下降，已自动发送关怀邮件，
        如客户回复请人工跟进"
    
    3. 7天后复查：
       IF 客户回复邮件 -> 通知CSM人工跟进
       IF 活跃度回升 -> 标记成功
       IF 无响应 -> 发送第二封邮件（加入限时优惠/免费培训）
    
    4. 再14天后终审：
       IF 仍无改善 -> 升级为 Level 3

---

Level 3 人工介入（系统辅助，L2级）：

  触发后：
    1. 向CSM发送紧急通知：
       ┌───────────────────────────────────────┐
       │ 需要人工介入：XX客户                    │
       │                                        │
       │ 健康分：28 | 连续下降42天                 │
       │ 自动干预历史：                           │
       │   • 3月1日 发送功能推荐邮件 -> 未响应     │
       │   • 3月8日 发送关怀邮件 -> 未响应         │
       │   • 3月22日 发送培训邀请 -> 未响应        │
       │ 合同到期：89天后                         │
       │ ARR: ￥8万                              │
       │                                        │
       │ 建议：电话联系确认客户是否仍在使用          │
       │ [一键生成电话脚本] [标记为流失]            │
       └───────────────────────────────────────┘
    
    2. CSM决策选项：
       a) 电话跟进 -> 系统提供话术建议
       b) 标记为流失风险 -> 启动最后挽回流程
       c) 标记为"低价值放弃" -> 到期不续费，释放精力
```

**环节四：效果验证与反馈**

```
周度自动汇总报告（推送至CSM）：

  ┌──────────────────────────────────────┐
  │  Tech-touch 周度运营报告               │
  │  时间：2025年第12周                    │
  │                                       │
  │  客户总数：215                         │
  │  健康分布：Green 168 | Yellow 35 | Red 12  │
  │                                       │
  │  本周自动干预：                         │
  │    Level 1 触发：8 次，成功挽回 5 例    │
  │    Level 2 触发：3 次，待观察 2 例      │
  │    Level 3 升级：1 例（XX公司，已通知）  │
  │                                       │
  │  自动邮件响应率：23%（行业均值18%）      │
  │  本周零需人工介入客户：96.3%            │
  │                                       │
  │  需要您关注：                           │
  │  1. XX公司 — Level 3，建议电话         │
  │  2. YY公司 — 续费45天倒计时，健康分52   │
  └──────────────────────────────────────┘

---

## Next Best Action (NBA) 每日行动引擎

> 融合 Gainsight CTA + Copilot 模式：不等 CSM 提问，主动推送每日个性化行动摘要。

### 设计理念

NBA 引擎是天枢中枢的"晨间简报"功能——每日 08:30 通过企微/钉钉 IM 推送，将多个 Skill 的输出聚合为一份可操作的行动清单。

### 数据汇聚源

| 数据源 | 提供 Skill | 内容 |
|--------|-----------|------|
| 活跃 Playbook 下一步 | Playbook Engine | 今天需推进的 Playbook 步骤 |
| 续约里程碑 | Renewal-War-Room | T-120/T-90/T-60/T-30 节点提醒 |
| 触点过期预警 | Health-Dashboard | 超过 X 天未互动的客户（按 tier 阈值） |
| 扩展信号 | Expansion-Radar | 置信度 > 70% 的新增信号 |
| Detractor 跟进 | Voice-of-Customer | 未关闭的 P0/P1 情感反馈 |
| 健康分骤降 | Churn-Predict | 过去 24h 新增 Red/Yellow 客户 |

### 推送模板

```
🌅 早安 {CSM_name}，以下是你今天的优先行动：

🔴 紧急 (P0-P1)
• [XX公司] 健康分昨日降至 38（-15），建议立即电话 → /health XX公司
• [YY公司] NPS 3分 Detractor 待处理（48h SLA 剩余 6h）→ /voc YY公司

🟡 重要 (P2)
• [ZZ公司] 续约倒计时 T-90，需启动策略确认 → /renewal ZZ公司
• [AA公司] 扩展信号：用量达合同 85%，置信度 82% → /expansion AA公司

📋 今日 Playbook 步骤
• PB-ONB-001 BB公司：第3步 关键用户培训（到期明天）
• PB-VAL-001 CC公司：QBR 数据准备（本周五前完成）

💡 洞察
• 你的客户池本周新增 2 个 Yellow 客户，建议关注
• AI 采纳率本周 62%（+5%），继续保持 👍
```

### 触发规则

- **推送时间**：工作日 08:30（时区跟随 CSM 设置）
- **推送渠道**：企微/钉钉（优先）> 邮件（备选）
- **自动化级别**：L1（纯信息展示，无需审批）
- **降级策略**：G1 = 部分数据缺失时仍推送可用项；G2 = 仅推送 Playbook 步骤和续约提醒

---

## Desired Business Outcome (DBO) 框架

> 融合 Gainsight Success Plans + Value Realization 最佳实践：从签约第一天起定义可衡量的业务成果。

### 客户 Frontmatter DBO 字段

```yaml
desired_outcomes:
  - outcome_id: "DBO-001"
    outcome_name: "客服工单处理效率提升"
    baseline_value: "平均处理时间 45 分钟"
    target_value: "平均处理时间 ≤ 20 分钟"
    current_value: "平均处理时间 28 分钟"
    measurement_method: "工单系统月度导出，取均值"
    status: "on_track"           # on_track | at_risk | achieved | abandoned
    target_date: "2026-06-30"
    last_measured: "2026-04-15"
  - outcome_id: "DBO-002"
    outcome_name: "客户自助服务率"
    baseline_value: "15%"
    target_value: "≥ 40%"
    current_value: "32%"
    measurement_method: "产品后台自助解决/总工单"
    status: "on_track"
    target_date: "2026-09-30"
    last_measured: "2026-04-15"
```

### DBO 与 Skill 联动

| 触发场景 | 关联 Skill | 行为 |
|---------|-----------|------|
| DBO 新建（签约时） | Sales-Handoff | 从合同/需求文档自动提取初始 DBO |
| DBO 进度更新 | Value-Proof-Generator | 自动计算 DBO 达成率，更新 current_value |
| DBO 达成 | Empathy-Writer | 生成恭喜信 + 案例邀请 |
| DBO at_risk | Health-Dashboard | 拉低 value_realization 维度评分 |
| QBR 准备 | Report-Builder | 自动将 DBO 进度纳入 QBR 报告 |

### DBO Check-in 节奏

- **Strategic 客户**：每月 DBO Review（纳入月度 touchpoint）
- **Growth 客户**：每季度 QBR 时 DBO 回顾
- **Scale 客户**：自动追踪，达成时自动通知
```
