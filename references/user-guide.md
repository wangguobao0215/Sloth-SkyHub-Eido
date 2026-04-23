# Sloth-SkyHub-Eido 用户指南

> 版本: 1.0.0

---

## 1. 技能定位

**Sloth-SkyHub-Eido**（智胜·天枢）是 Sloth-Eido 技能家族中的 **客户成功智能体矩阵系统**。它以客户旅程为骨架、价值实现为内核，编排 17 个专业子 Skill，覆盖从销售交接到续约扩展的客户全生命周期。

核心价值：

- **旅程驱动** -- 6 个生命周期阶段，系统根据客户当前阶段自动调整可用能力
- **17 个专业子 Skill** -- 独立运行、协同增强，覆盖健康评分、流失预测、续约管理等全场景
- **三层客户分级** -- Strategic / Growth / Scale，每层 AI 角色自动适配
- **Playbook 引擎** -- 10 个核心剧本，触发条件 -> 分步执行 -> 分支判断 -> 升级路径
- **本地优先** -- 基于 Obsidian/Markdown 知识库，数据主权完全可控
- **优雅降级** -- 数据不全时不停摆，降级输出并标注补全路径
- **中国市场深度适配** -- IM-first、关系型续约、企微/钉钉话术、国企合规

适用人群：CSM 团队负责人、客户成功经理、客户成功运营人员。

---

## 2. 安装与初始化

### 2.1 安装

将 `Sloth-SkyHub-Eido` 文件夹复制到 QoderWork 技能目录：

```bash
cp -r Sloth-SkyHub-Eido ~/.qoderwork/skills/
```

### 2.2 初始化知识库

在 Obsidian 中创建 `SkyHub-KB` Vault，按以下标准目录结构初始化：

```
vault/
  customers/        # 客户档案
  playbooks/        # Playbook 实例
  templates/        # 输出模板
  health-snapshots/ # 健康评分快照
  meeting-notes/    # 会议记录
```

详细的目录规范、文件命名和元数据标准参见 [obsidian-vault.md](obsidian-vault.md)。

### 2.3 导入客户数据

将现有客户信息（Excel/CSV/CRM 导出）导入到 `customers/` 目录。首次使用时，技能会引导你完成数据导入。

### 2.4 首次使用引导

首次触发 SkyHub 时，技能会自动执行引导流程：

1. 欢迎与介绍
2. 数据导入确认
3. 知识库初始化检查
4. 推荐起步路径（KB-QA 或 Health-Dashboard）
5. 解释 4 期采纳旅程（P0-P3）

---

## 3. 核心功能与命令

### 3.1 子 Skill 路由表

| 命令关键词 | 匹配子 Skill | 旅程阶段 | 说明 |
|-----------|-------------|---------|------|
| `交接` `新签` `新客户` `签约` | Sales-Handoff | Onboarding | 销售-CS 交接清单与风险检查 |
| `上线进度` `TTV` `实施` `部署` | TTV-Tracker | Onboarding | 首次价值实现路径追踪 |
| `采纳` `使用率` `活跃度` `DAU` `MAU` | Adoption-Analyzer | Adoption | 产品采纳分析与干预建议 |
| `价值` `ROI` `QBR` `EBR` `业务成果` | Value-Proof-Generator | Value Realization | 价值证明文档与 QBR 报告生成 |
| `客户健康` `健康分` `风险` `预警` | Health-Dashboard | 全阶段 | 多维健康评分与趋势分析 |
| `流失` `churn` `预警` `预测` | Churn-Predict | Growth/Renewal | 流失风险预测与干预建议 |
| `扩展` `增购` `upsell` `cross-sell` | Expansion-Radar | Growth | 扩展机会识别与推进策略 |
| `竞品` `竞对` `对手` `替代品` | Competitor-Radar | Growth/Renewal | 竞品渗透监控与防御策略 |
| `续约` `到期` `renew` `合同` | Renewal-War-Room | Renewal | 续约作战室与谈判策略 |
| `话术` `回复` `措辞` `沟通` `邮件` | Empathy-Writer | 全阶段 | 场景化沟通内容生成 |
| `报告` `PPT` `汇报` `总结` | Report-Builder | 全阶段 | 结构化报告与 PPT 生成 |
| `审批` `流程` `工单` | Workflow-Agent | 全阶段 | 内部流程自动化与审批跟踪 |
| `故障` `报错` `日志` `工单升级` | Log-Analyzer | 全阶段 | 技术日志分析与问题定位 |
| `陪练` `模拟` `练习` `场景演练` | Role-Play-Coach | 全阶段 | CSM 技能陪练与场景模拟 |
| `高管` `简报` `executive` `CXO` | Executive-Briefing | 全阶段 | 高管级客户简报生成 |
| `知识` `文档` `怎么` `如何` `FAQ` | KB-QA | 全阶段 | 知识库检索与智能问答 |
| `NPS` `满意度` `情绪` `投诉` `CSAT` | Voice-of-Customer | 全阶段 | 客户之声采集与情绪分析 |

### 3.2 元功能命令

| 命令关键词 | 路由目标 | 说明 |
|-----------|---------|------|
| `Playbook` `剧本` `流程启动` `触发` | Playbook 引擎 | 启动/查看/管理 Playbook 实例 |
| `仪表盘` `看板` `dashboard` `总览` | Dashboard 聚合查询 | 跨 Skill 数据聚合展示 |
| `切换客户` `换一个客户` | 客户切换 | 更新当前 customer_id |
| `批量` `全部客户` `客户列表` | 批量模式 | 多客户维度分析与操作 |

---

## 4. 工作流程指南

### 4.1 新客户接入流程

1. **触发**: 销售告知有新签客户
2. **交接**: 发送"新客户交接" -> Sales-Handoff 生成交接清单
3. **建档**: 系统自动在知识库创建客户档案
4. **TTV 追踪**: TTV-Tracker 自动建立首次价值实现路径
5. **健康基线**: Health-Dashboard 建立初始健康评分

### 4.2 日常客户管理流程

1. **查看健康**: "看一下 XX 客户的健康分" -> Health-Dashboard
2. **分析采纳**: "XX 客户的使用率怎么样" -> Adoption-Analyzer
3. **准备沟通**: "帮我写一封跟进邮件" -> Empathy-Writer
4. **生成报告**: "给 XX 客户做 QBR 报告" -> Value-Proof-Generator + Report-Builder

### 4.3 续约管理流程

1. **提前预警**: 合同到期前 120 天，Renewal-War-Room 自动激活
2. **风险评估**: Churn-Predict 输出流失风险因子排序
3. **价值论据**: Value-Proof-Generator 准备 ROI 证明材料
4. **竞品防御**: Competitor-Radar 监控竞品渗透信号
5. **谈判策略**: Renewal-War-Room 生成定价参考和谈判策略
6. **高管简报**: Executive-Briefing 为 CXO 级汇报准备材料

### 4.4 复合意图处理

当你的请求同时涉及多个 Skill 时，系统会自动分解：

```
用户: 帮我看一下 XX 客户的健康分和续约风险
系统: 你提到了两件事。我先拉取健康评分，再基于评分做流失预测，可以吗？
```

上游输出自动传递到下游作为输入上下文。

---

## 5. 关键系统说明

### 5.1 客户旅程六阶段

| 阶段 | 时间窗口 | 核心目标 | 激活 Skill |
|------|---------|---------|-----------|
| 交接上手 | Day 0-14 | 无缝交接，建立信任 | Sales-Handoff, TTV-Tracker |
| 首次价值 | Day 15-90 | 达成 TTV | TTV-Tracker, Adoption-Analyzer, KB-QA |
| 深度采纳 | Month 3-9 | 扩大使用深度和广度 | Adoption-Analyzer, Value-Proof-Generator |
| 价值成长 | Month 9-18 | 发现扩展机会 | Expansion-Radar, Value-Proof-Generator |
| 续约决策 | 到期前 120-30 天 | 确保续约 | Renewal-War-Room, Churn-Predict |
| 持续伙伴 | 续约后 | 深化合作 | Voice-of-Customer, Expansion-Radar |

详见 [customer-journey.md](customer-journey.md)。

### 5.2 三层客户分级

| 层级 | 典型特征 | AI 角色 | 服务模式 |
|------|---------|---------|---------|
| Strategic | ARR Top 20%，决策链长 | 幕后军师 | High-touch: AI 准备材料，CSM 亲自交付 |
| Growth | ARR 中段，标准需求 | 副驾驶 | Mid-touch: AI 起草初稿，CSM 审阅执行 |
| Scale | ARR 长尾，标准化场景 | 主角 | Tech-touch: AI 自动执行低风险操作 |

详见 [tier-strategy.md](tier-strategy.md)。

### 5.3 健康评分模型

7 维度加权评分（总分 0-100）：产品采纳 25% / 支持健康 15% / 合同状态 15% / 关系深度 15% / 价值实现 15% / 客户情绪 10% / 竞争环境 5%。

阈值：绿色 >= 70 / 黄色 40-69 / 红色 < 40。权重按旅程阶段动态调整。

详见 [health-score-model.md](health-score-model.md)。

### 5.4 优雅降级机制

| 级别 | 触发条件 | 行为 |
|------|---------|------|
| G1 精度降低 | 部分数据缺失 | 用已有数据计算，标注缺失维度和精度影响 |
| G2 维度缺失 | 关键数据源不可用 | 跳过不可用维度，标注可用维度范围 |
| G3 仅骨架 | 几乎无数据 | 输出标准模板骨架 + CSM 手动填充指引 |

---

## 6. 最佳实践

1. **从 P0 单点突破开始** -- 先激活 KB-QA 和 Health-Dashboard 两个 Skill，积累数据后再逐步扩展到更多 Skill。不要试图一次激活全部 17 个 Skill。
2. **坚持录入数据** -- SkyHub 的价值与数据完整度成正比。每次客户互动后及时录入会议记录、健康信号等信息。
3. **善用 Playbook** -- 标准化流程用 Playbook 驱动，比手动执行更不容易遗漏关键步骤。
4. **关注健康分趋势而非绝对值** -- 单次健康分只是快照，连续趋势才能真正反映客户状态变化。
5. **让 AI 准备、你来交付** -- 对于 Strategic 层客户，AI 生成的材料始终需要 CSM 审阅和个性化调整后再交付给客户。
6. **利用跨 Skill 数据流** -- Health-Dashboard 的评分会自动流入 Churn-Predict 和 Renewal-War-Room，充分利用这些自动关联减少重复工作。

---

## 7. 常见问题 (FAQ)

### Q1: 我没有现成的客户数据，能用 SkyHub 吗？

可以。SkyHub 内置优雅降级机制，数据不全时会降级输出并标注补全路径。你可以从零开始，手动录入客户信息，逐步积累数据。建议先从一个客户开始试用。

### Q2: 17 个子 Skill 需要全部激活吗？

不需要。建议按 4 期实施路线逐步启用：P0 先用 KB-QA + Health-Dashboard，P1 加入 Playbook + 交接 + TTV，P2 加入 Adoption + Churn + Expansion，P3 全部激活。

### Q3: 客户分层是自动判定的吗？

是的。系统根据 ARR、战略价值和复杂度自动判定客户层级（Strategic / Growth / Scale），并据此调整 AI 的角色和自动化水平。你也可以手动覆盖分层结果。

### Q4: 健康评分的权重可以调整吗？

默认权重按旅程阶段动态调整（如 Onboarding 阶段"产品采纳"权重上调至 35%）。如果需要自定义权重，可以在 [health-score-model.md](health-score-model.md) 中修改配置。

### Q5: SkyHub 会自动发送邮件给客户吗？

取决于自动化等级。L1-L2 级别下所有外部触达需 CSM 确认；L3 级别下邮件/消息需 CSM 审阅后发送；L4-L5 级别下低风险操作可自动执行。系统永远不会超过客户被分配的自动化等级。

---

## 8. 参考文档索引

| 文档 | 说明 |
|------|------|
| [customer-journey.md](customer-journey.md) | 客户旅程六阶段详解 -- 阶段定义、转换信号、异常处理 |
| [tier-strategy.md](tier-strategy.md) | 三层客户分级策略 -- 判定规则、跨层迁移、自动化上限 |
| [skill-matrix.md](skill-matrix.md) | 17 个子 Skill 详细规格 -- 输入/输出、触发逻辑、降级策略 |
| [health-score-model.md](health-score-model.md) | 健康评分模型详解 -- 评分公式、维度细分、阶段权重矩阵 |
| [playbook-engine.md](playbook-engine.md) | Playbook 引擎与 10 个核心剧本 -- 触发规则、步骤模板、分支逻辑 |
| [metrics-kpi.md](metrics-kpi.md) | 度量体系与 KPI -- 北极星指标、过程指标、效率指标 |
| [security-governance.md](security-governance.md) | 安全、治理与合规 -- 数据分级、自动化管控、审计规范 |
| [obsidian-vault.md](obsidian-vault.md) | Obsidian 知识库结构 -- 目录规范、文件命名、元数据标准 |
| [mcp-integration.md](mcp-integration.md) | MCP 集成架构 -- 外部数据源对接、API 映射 |
| [implementation-roadmap.md](implementation-roadmap.md) | 四期实施路线图 -- P0-P3 分期目标与里程碑 |
| [risk-matrix.md](risk-matrix.md) | 风险矩阵与缓解措施 -- 技术风险、运营风险、组织风险 |
| [skill-integration.md](skill-integration.md) | 跨 Skill 协同与数据流 -- 完整拓扑图、依赖链、冲突解决 |
