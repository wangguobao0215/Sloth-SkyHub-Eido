# SkyHub 子Skill完整规格手册

> 本文档定义 SkyHub 系统全部 17 个子 Skill 的阶段激活矩阵与详细规格卡。
> 所有 Skill 遵循统一的分层行为（Strategic / Growth / Scale）和降级策略（G1-G3）框架。

---

## 一、阶段激活矩阵

下表标注每个 Skill 在客户旅程七阶段中的激活状态（`*` = 激活）。

| # | Skill | Handoff | Onboarding | Adoption | Value Realization | Expansion | Renewal | Advocacy |
|---|-------|:-------:|:----------:|:--------:|:-----------------:|:---------:|:-------:|:--------:|
| 1 | Health-Dashboard | | * | * | * | * | * | |
| 2 | KB-QA | * | * | * | * | * | * | |
| 3 | Voice-of-Customer | | * | * | * | * | * | * |
| 4 | Sales-Handoff | * | | | | | | |
| 5 | TTV-Tracker | | * | | | | | |
| 6 | Adoption-Analyzer | | | * | * | | | |
| 7 | Value-Proof-Generator | | | | * | * | * | * |
| 8 | Expansion-Radar | | | * | * | * | | |
| 9 | Competitor-Radar | | | | | * | * | |
| 10 | Renewal-War-Room | | | | | | * | |
| 11 | Churn-Predict | | | * | | | * | |
| 12 | Empathy-Writer | * | * | * | * | * | * | * |
| 13 | Report-Builder | | | | * | | * | |
| 14 | Workflow-Agent | | * | | | * | * | |
| 15 | Log-Analyzer | | * | * | | | | |
| 16 | Role-Play-Coach | | | * | | | * | |
| 17 | Executive-Briefing | | | | * | * | * | * |

---

## 二、Skill 详细规格卡

---

### Skill 1: 客户健康度仪表盘 (Health-Dashboard)

| 字段 | 值 |
|------|-----|
| 适用阶段 | 全阶段（权重随阶段动态调整） |
| 自动化级别 | L2（AI 计算分析，CSM 审核决策） |
| 状态 | active |

**分层行为**
- **Strategic**: 7 维深度报告 + 趋势分析 + 行动建议；AI = 分析师，CSM = 决策者
- **Growth**: 标准健康报告 + 风险高亮 + Top 3 建议；AI = 副驾驶，CSM 快速审核
- **Scale**: 仅评分计算 + 自动触发警报；无 CSM 审核，异常自动升级

**核心功能**
整合 7 个维度（usage_depth, activity_trend, commercial_health, support_tickets, stakeholder_engagement, value_realization, sentiment）通过规则引擎计算健康分（0-100）。输出健康卡片、风险预警、行动建议。维度权重按旅程阶段自动调整。

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| customer_id | string | 是 | 客户唯一标识 |
| depth | enum | 否 | quick / standard / deep，默认 standard |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| health_card | markdown | 健康卡片摘要 |
| risk_alerts | list | 风险预警列表 |
| action_recommendations | list | 行动建议 |
| frontmatter_update | yaml | 客户档案健康字段更新 |

**数据源**: Obsidian 客户文件（frontmatter）, 本地 CSV/Excel（产品数据）, CRM API（合同/账单 via MCP）

**降级策略**
- G1: 使用缓存分数，注明"数据截至 [日期]"
- G2: 仅计算可用维度，注明缺失项
- G3: 返回原始数据 + "待补充数据"清单

**协同关系**
- 依赖 <-: KB-QA
- 触发 ->: Churn-Predict, Renewal-War-Room
- 被调用 <-: Report-Builder, Executive-Briefing

**触发关键词**: 客户健康, 健康分, 健康度, 风险评估, health score, 体检

---

### Skill 2: 知识库问答 (KB-QA)

| 字段 | 值 |
|------|-----|
| 适用阶段 | Handoff -> Renewal（全旅程通用） |
| 自动化级别 | L4（全自动回答，置信度低时升级 CSM） |
| 状态 | active |

**分层行为**
- **Strategic**: 精准检索 + 上下文增强回答 + 关联推荐 + 自动生成跟进建议；CSM 可定制知识库范围
- **Growth**: 标准 RAG 检索 + 直接回答 + 来源引用；低置信度自动升级
- **Scale**: 仅 FAQ 匹配 + 标准回答模板；无匹配时返回工单链接

**核心功能**
基于 Obsidian Vault 的语义搜索引擎。对客户问题进行意图识别，在产品文档、历史工单、最佳实践库中检索最相关内容，通过 RAG（Retrieval-Augmented Generation）生成带引用来源的回答。支持多轮对话追问，自动记录知识缺口并反馈至知识运营。

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| question | string | 是 | 用户问题（自然语言） |
| customer_id | string | 否 | 客户标识，用于上下文增强（如已知特定版本/配置） |
| scope | enum | 否 | all / product / best_practice / ticket，默认 all |
| lang | enum | 否 | zh / en，默认 zh |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| answer | markdown | 生成的回答正文 |
| sources | list | 引用来源列表（文件路径 + 相关段落） |
| confidence | float | 回答置信度（0.0-1.0） |
| follow_up_suggestions | list | 建议追问的相关问题 |
| knowledge_gap_flag | boolean | 是否检测到知识缺口 |

**数据源**: Obsidian Vault（产品文档、FAQ、最佳实践）, 历史工单库, 客户配置文件（frontmatter）

**降级策略**
- G1: 仅返回最相关文档片段，不做生成式回答
- G2: 返回关键词匹配结果（退回 BM25），注明"语义搜索不可用"
- G3: 返回"未找到相关内容" + 建议联系支持团队

**协同关系**
- 依赖 <-: 无（基础设施级 Skill）
- 触发 ->: Log-Analyzer（技术问题自动关联）
- 被调用 <-: Health-Dashboard, Empathy-Writer, Report-Builder, Role-Play-Coach, 几乎所有 Skill

**触发关键词**: 知识库, 搜索, 查一下, 怎么用, 如何配置, FAQ, 文档, KB, knowledge base

---

### Skill 3: 客户之声 (Voice-of-Customer)

| 字段 | 值 |
|------|-----|
| 适用阶段 | Onboarding -> Renewal（持续采集） |
| 自动化级别 | L2（AI 采集分类，CSM 审核解读） |
| 状态 | active |

**分层行为**
- **Strategic**: 全渠道深度情感分析 + 8 类分类 + P0-P3 分级 + 趋势报告 + 行动建议；CSM 每周 review
- **Growth**: 关键渠道情感分析 + 分类 + 高优先级自动通知；CSM 仅处理 P0/P1
- **Scale**: 仅 NPS/CSAT 评分采集 + 负面自动告警；无主动分析

**核心功能**
从 IM 聊天记录、NPS 调研、CSAT 评分、工单评价、会议纪要等多渠道采集客户反馈，进行情感分析（positive / neutral / negative / mixed）和 8 类主题分类（product_bug, feature_request, usability, performance, pricing, support_quality, onboarding, general_satisfaction）。按 P0-P3 分级（P0 = 流失风险级，P3 = 一般反馈），生成 VoC 周报/月报。

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| customer_id | string | 是 | 客户唯一标识 |
| source | enum | 否 | im / nps / csat / ticket / meeting / all，默认 all |
| time_range | string | 否 | 时间范围，如 7d / 30d / 90d，默认 30d |
| priority_filter | enum | 否 | P0 / P1 / P2 / P3 / all，默认 all |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| feedback_items | list | 反馈条目列表（含来源、情感、分类、优先级） |
| sentiment_summary | object | 情感分布统计（positive/neutral/negative/mixed 占比） |
| category_distribution | object | 8 类主题分布 |
| priority_breakdown | object | P0-P3 分布 |
| trend_analysis | markdown | 情感趋势分析（与上期对比） |
| action_items | list | 建议行动项 |

**数据源**: IM 聊天导出文件, NPS/CSAT 调研数据（CSV）, 工单系统, 会议纪要（Obsidian）

**降级策略**
- G1: 仅分析可用渠道数据，注明缺失渠道
- G2: 跳过情感分析，仅做关键词分类
- G3: 返回原始反馈列表，无分析

**协同关系**
- 依赖 <-: KB-QA（反馈内容关联知识库）
- 触发 ->: Health-Dashboard（情感维度更新）, Churn-Predict（负面信号）
- 被调用 <-: Report-Builder, Executive-Briefing

**触发关键词**: 客户之声, VoC, 反馈, 情感分析, NPS, CSAT, 满意度, 客户情绪

---

### Skill 4: 销售交接 (Sales-Handoff)

| 字段 | 值 |
|------|-----|
| 适用阶段 | Handoff（仅限交接阶段） |
| 自动化级别 | L2（AI 提取整理，CSM 审核确认） |
| 状态 | active |

**分层行为**
- **Strategic**: 完整交接报告 + 承诺清单校验 + 干系人图谱 + 30-60-90 天计划 + 风险预判；CSM 与 AE 联合 review
- **Growth**: 标准交接模板 + 承诺清单 + 基础 30-60-90 计划；CSM 快速确认
- **Scale**: 精简交接卡片 + 关键承诺列表；自动初始化客户档案

**核心功能**
从销售 CRM 记录、合同条款、售前沟通记录中提取关键信息，生成结构化交接报告。核心包括：(1) 承诺验证 -- 逐条比对销售承诺与合同条款，标记差异和风险承诺；(2) 客户档案初始化 -- 自动创建 Obsidian 客户文件并填充 frontmatter；(3) 30-60-90 天计划 -- 基于客户画像和产品复杂度自动生成分阶段落地计划。

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| deal_id | string | 是 | 销售机会/合同编号 |
| sales_notes | markdown | 否 | 销售侧补充说明 |
| contract_summary | markdown | 否 | 合同摘要（如无则从 CRM 拉取） |
| customer_tier | enum | 否 | strategic / growth / scale，默认由 ARR 自动判定 |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| handoff_report | markdown | 完整交接报告 |
| promise_checklist | list | 承诺清单（含状态：confirmed / at_risk / gap） |
| customer_profile | yaml | 初始化的客户档案 frontmatter |
| onboarding_plan | markdown | 30-60-90 天落地计划 |
| risk_flags | list | 交接风险标记 |
| stakeholder_map | object | 干系人图谱（角色 + 影响力 + 态度） |

**数据源**: CRM API（合同、销售记录 via MCP）, 售前沟通记录, 合同文档

**降级策略**
- G1: 使用 CSM 手动填写模板，AI 辅助格式化
- G2: 仅生成基础档案卡片，承诺清单留空待补
- G3: 创建空白客户档案模板 + 待办清单

**协同关系**
- 依赖 <-: 无
- 触发 ->: TTV-Tracker（启动 Onboarding 跟踪）, Health-Dashboard（初始化基线）
- 被调用 <-: 无（入口级 Skill）

**触发关键词**: 交接, 新客户, 销售交接, handoff, 接手, 新签, 合同签约

---

### Skill 5: TTV 追踪器 (TTV-Tracker)

| 字段 | 值 |
|------|-----|
| 适用阶段 | Onboarding（首次价值实现前） |
| 自动化级别 | L3（AI 自动追踪预测，CSM 处理偏差） |
| 状态 | active |

**分层行为**
- **Strategic**: 自定义里程碑体系 + 实时进度仪表盘 + TTV 预测模型 + 偏差根因分析 + 行动建议；CSM 每日关注
- **Growth**: 标准里程碑模板 + 进度追踪 + 偏差告警；CSM 每周 review
- **Scale**: 简化里程碑（3-5 个关键节点）+ 自动进度检测 + 超时告警

**核心功能**
定义并追踪客户从签约到首次价值实现（Time-to-Value）的关键里程碑。基于客户画像和产品复杂度预估 TTV 基线，实时监控里程碑完成进度，当检测到偏差（延迟、跳过、阻塞）时自动告警并分析原因。里程碑模板可自定义，默认包括：环境就绪、数据导入、核心配置、首次使用、首次价值事件。

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| customer_id | string | 是 | 客户唯一标识 |
| milestone_template | enum | 否 | default / saas_standard / complex_deploy / custom，默认 default |
| target_ttv_days | int | 否 | 目标 TTV 天数（如无则自动预估） |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| milestone_status | list | 各里程碑状态（completed / in_progress / blocked / not_started） |
| ttv_prediction | object | 预测 TTV（天数 + 置信度 + 风险因素） |
| deviation_alerts | list | 偏差告警（延迟天数 + 根因分析） |
| progress_dashboard | markdown | 进度仪表盘（可视化） |
| next_actions | list | 下一步行动建议 |

**数据源**: Obsidian 客户文件（里程碑记录）, 产品使用日志（CSV）, 会议纪要

**降级策略**
- G1: 基于历史同类客户平均 TTV 预估，注明"基于统计预估"
- G2: 仅追踪手动更新的里程碑，不做自动检测
- G3: 返回里程碑 checklist 模板，由 CSM 手动更新

**协同关系**
- 依赖 <-: Sales-Handoff（初始化里程碑基线）
- 触发 ->: Health-Dashboard（Onboarding 阶段健康度）, Adoption-Analyzer（完成 Onboarding 后衔接）
- 被调用 <-: Report-Builder

**触发关键词**: TTV, 上线进度, 里程碑, 实施进度, onboarding 进度, 首次价值, 落地追踪

---

### Skill 6: 采纳度分析器 (Adoption-Analyzer)

| 字段 | 值 |
|------|-----|
| 适用阶段 | Adoption -> Value Realization |
| 自动化级别 | L2（AI 分析洞察，CSM 制定干预方案） |
| 状态 | active |

**分层行为**
- **Strategic**: 多维采纳度深度报告 + 功能渗透热力图 + 沉默用户画像 + 个性化激活方案；CSM 主导干预
- **Growth**: 标准采纳度报告 + DAU/MAU 趋势 + Top 沉默用户列表；CSM 选择性干预
- **Scale**: 仅 DAU/MAU 指标 + 沉默客户自动触发邮件/消息；无 CSM 介入

**核心功能**
分析客户的产品功能渗透率（Feature Penetration Rate），追踪 DAU/MAU 活跃度趋势，检测沉默客户（定义：连续 N 天无登录或核心功能使用为零）。输出功能采纳热力图、用户活跃度排行、沉默用户预警。帮助 CSM 精准定位"买了但不用"的功能模块，制定针对性激活策略。

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| customer_id | string | 是 | 客户唯一标识 |
| time_range | string | 否 | 分析时间范围，如 7d / 30d / 90d，默认 30d |
| silent_threshold_days | int | 否 | 沉默客户阈值天数，默认 14 |
| feature_list | list | 否 | 关注的功能列表（如无则分析全部） |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| adoption_summary | markdown | 采纳度总览 |
| feature_penetration | object | 功能渗透率（功能名 -> 使用率%） |
| dau_mau_trend | object | DAU/MAU 趋势数据 |
| silent_users | list | 沉默用户/账号列表（含最后活跃时间） |
| activation_recommendations | list | 激活建议（按功能模块） |
| heatmap | markdown | 功能使用热力图（文本可视化） |

**数据源**: 产品使用日志（CSV/Excel）, Obsidian 客户文件, 功能清单配置

**降级策略**
- G1: 使用上期数据对比，注明"当前期数据不完整"
- G2: 仅计算登录活跃度，跳过功能维度分析
- G3: 返回原始使用数据摘要 + "需人工分析"标记

**协同关系**
- 依赖 <-: TTV-Tracker（Onboarding 完成后启动）
- 触发 ->: Health-Dashboard（usage_depth 维度）, Expansion-Radar（高采纳信号）, Churn-Predict（低采纳信号）
- 被调用 <-: Report-Builder, Value-Proof-Generator

**触发关键词**: 采纳度, 使用率, 功能渗透, DAU, MAU, 沉默客户, 活跃度, 用了哪些功能

---

### Skill 7: 价值证明生成器 (Value-Proof-Generator)

| 字段 | 值 |
|------|-----|
| 适用阶段 | Value Realization -> Renewal（SkyHub 最核心 Skill） |
| 自动化级别 | L3（AI 自动计算生成，CSM 校准叙事） |
| 状态 | active |

**分层行为**
- **Strategic**: 定制 ROI 模型 + 多维价值叙事 + 行业基准对比 + C-level 价值摘要 + 案例故事化包装；CSM 深度参与叙事打磨
- **Growth**: 标准 ROI 计算 + 价值摘要卡片 + 基础基准对比；CSM 审核数据准确性
- **Scale**: 自动 ROI 计算 + 模板化价值报告；无 CSM 干预

**核心功能**
SkyHub 系统最核心的 Skill。负责将客户使用数据转化为可量化的价值证明。三大核心能力：(1) ROI 计算引擎 -- 基于使用数据、效率提升、成本节约等维度计算投资回报率，支持直接价值（可量化）和间接价值（定性描述）；(2) 价值叙事生成 -- 将冰冷的数据包装成有说服力的故事（"从 X 到 Y，带来了 Z"），适配不同受众（技术负责人 / 业务负责人 / C-level）；(3) 行业基准对比 -- 将客户表现与同行业、同规模客户进行对标。这是续约、增购的弹药库。

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| customer_id | string | 是 | 客户唯一标识 |
| value_dimensions | list | 否 | 关注的价值维度（efficiency / cost_saving / revenue_growth / risk_reduction / compliance），默认全部 |
| audience | enum | 否 | tech_lead / biz_lead / c_level / all，默认 all |
| benchmark_scope | enum | 否 | industry / size / region / none，默认 industry |
| time_range | string | 否 | 价值核算周期，如 90d / 180d / 1y，默认 90d |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| roi_calculation | object | ROI 计算结果（投入、产出、ROI%、回收期） |
| value_narrative | markdown | 价值叙事报告（按受众定制） |
| benchmark_comparison | object | 行业基准对比（百分位排名 + 差距分析） |
| value_highlights | list | Top 3 价值亮点（一句话总结） |
| data_sources_used | list | 使用的数据源清单（透明度） |
| confidence_level | enum | 价值计算置信度（high / medium / low） |
| value_card | markdown | 一页纸价值摘要卡片 |

**数据源**: 产品使用数据（CSV/Excel）, 客户业务指标（手动输入或 CRM）, 行业基准库（Obsidian）, 合同金额（CRM API）

**降级策略**
- G1: 使用行业平均值替代缺失数据，注明"部分基于行业平均估算"
- G2: 仅生成定性价值描述，跳过定量 ROI 计算
- G3: 返回价值证明模板 + 数据收集清单

**协同关系**
- 依赖 <-: Adoption-Analyzer（使用数据）, Health-Dashboard（健康趋势）
- 触发 ->: Renewal-War-Room（续约价值弹药）, Expansion-Radar（增购价值论证）, Executive-Briefing（高管汇报素材）
- 被调用 <-: Report-Builder, Empathy-Writer

**触发关键词**: 价值证明, ROI, 投资回报, 价值报告, 续约价值, 客户价值, value proof, 效果评估, 成果展示

---

### Skill 8: 增购雷达 (Expansion-Radar)

| 字段 | 值 |
|------|-----|
| 适用阶段 | Adoption -> Expansion |
| 自动化级别 | L2（AI 信号检测评分，CSM 判断跟进策略） |
| 状态 | active |

**核心功能**
实时监测客户的增购信号，从多维度捕捉扩展机会。三大信号源：(1) 用量信号 -- 核心功能使用率 >80%、API 调用量接近配额上限、存储/席位即将用尽；(2) 行为信号 -- 客户主动询问未购买功能、频繁访问高级功能文档、参加产品 Webinar；(3) 组织信号 -- 新部门开始使用产品、新增管理员账号、跨区域访问增加。对每个信号进行置信度评分（0-100），综合评估增购可能性。

**分层行为**
- **Strategic**: 全维度信号扫描 + 置信度评分 + 增购方案建议 + 预估增购金额 + 最佳切入时机；CSM 制定增购策略
- **Growth**: 关键信号监测 + 置信度评分 + Top 3 增购机会；CSM 选择跟进
- **Scale**: 仅用量阈值告警（>80%）+ 自动发送升级提示邮件

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| customer_id | string | 是 | 客户唯一标识 |
| signal_types | list | 否 | usage / behavior / organization / all，默认 all |
| threshold | int | 否 | 最低置信度阈值（0-100），默认 60 |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| expansion_signals | list | 增购信号列表（类型、描述、置信度、数据来源） |
| opportunity_score | int | 综合增购可能性评分（0-100） |
| recommended_products | list | 建议增购产品/模块 |
| estimated_value | object | 预估增购金额范围 |
| timing_suggestion | string | 建议切入时机 |
| talk_track | markdown | 增购话术建议 |

**数据源**: 产品使用日志（CSV/Excel）, CRM 合同数据（当前购买清单）, 客户行为日志, Obsidian 客户文件

**降级策略**
- G1: 仅分析用量信号（数据最可靠），跳过行为和组织信号
- G2: 基于合同到期日 + 用量趋势做简单预判
- G3: 返回当前用量报告 + 产品目录对照表

**协同关系**
- 依赖 <-: Adoption-Analyzer（使用数据）, Health-Dashboard（健康度背景）
- 触发 ->: Value-Proof-Generator（增购价值论证）, Empathy-Writer（增购沟通邮件）
- 被调用 <-: Report-Builder, Executive-Briefing

**触发关键词**: 增购, 扩展, 升级, upsell, cross-sell, 用量快满了, 新部门, 加购

---

### Skill 9: 竞对雷达 (Competitor-Radar)

| 字段 | 值 |
|------|-----|
| 适用阶段 | Expansion -> Renewal |
| 自动化级别 | L2（AI 情报整理，CSM 制定应对策略） |
| 状态 | active |

**分层行为**
- **Strategic**: 深度竞对分析报告 + 差异化话术 + 切换成本计算 + 攻防策略 + 定制 battle card；CSM 主导竞争应对
- **Growth**: 标准 battle card + 关键差异点 + 切换成本概要；CSM 按需使用
- **Scale**: 简化竞对对比表 + 标准化差异话术

**核心功能**
维护竞对情报库，为 CSM 提供竞争应对弹药。三大核心能力：(1) Win/Loss 案例库 -- 从历史竞争案例中提取成功/失败因素，按行业、规模、场景分类；(2) 差异化话术生成 -- 针对特定竞对自动生成差异化定位话术，强调我方优势、规避劣势；(3) 切换成本分析 -- 量化客户从我方切换至竞对的直接成本（迁移、重新培训、定制丢失）和间接成本（风险、时间、机会成本），构建"切换护城河"。

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| competitor_name | string | 是 | 竞对名称 |
| customer_id | string | 否 | 客户标识（用于定制化分析） |
| scenario | enum | 否 | defensive（防守）/ offensive（进攻）/ comparison（对比），默认 defensive |
| focus_areas | list | 否 | 关注的对比维度（price / feature / service / ecosystem / security） |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| battle_card | markdown | 竞对 battle card（攻防两面） |
| differentiators | list | 关键差异点（我方优势 + 竞对劣势） |
| switching_cost_analysis | object | 切换成本分析（直接成本 + 间接成本 + 总计） |
| win_loss_cases | list | 相关 Win/Loss 案例摘要 |
| talk_tracks | list | 场景化话术（按 focus_area 分组） |
| risk_assessment | markdown | 竞争风险评估 |

**数据源**: Obsidian 竞对情报库, Win/Loss 案例库, 产品功能对比表, 行业分析报告

**降级策略**
- G1: 使用通用竞对模板，注明"需补充最新情报"
- G2: 仅返回功能对比表，无话术生成
- G3: 返回竞对情报收集模板 + 信息需求清单

**协同关系**
- 依赖 <-: KB-QA（产品知识支撑）
- 触发 ->: Renewal-War-Room（续约竞争应对）, Empathy-Writer（竞争应对邮件）
- 被调用 <-: Report-Builder, Executive-Briefing, Role-Play-Coach

**触发关键词**: 竞对, 竞争, 对手, battle card, 差异化, 切换成本, 友商, 竞品

---

### Skill 10: 续约作战室 (Renewal-War-Room)

| 字段 | 值 |
|------|-----|
| 适用阶段 | Renewal（120 天倒计时启动） |
| 自动化级别 | L2（AI 全面分析准备，CSM 主导谈判执行） |
| 状态 | active |

**分层行为**
- **Strategic**: 120 天完整作战计划 + 多维风险评估 + 定价策略建议 + 价值一页纸 + 干系人攻略 + 招标/关系策略（中国特色）；CSM 全程主导
- **Growth**: 90 天标准续约流程 + 风险评估 + 价值摘要 + 定价建议；CSM 按流程推进
- **Scale**: 60 天自动续约提醒 + 简化价值卡片 + 标准报价；自动化流程为主

**核心功能**
续约阶段的综合作战指挥中心，以 120 天倒计时为节奏驱动全流程。核心模块：(1) 续约风险评估 -- 综合健康分、NPS、使用趋势、竞对情报、干系人变动等因素评估续约风险（red / yellow / green）；(2) 定价建议 -- 基于客户价值实现程度、市场竞争、合同历史给出涨价/持平/折扣建议；(3) 价值一页纸 -- 自动生成面向客户决策者的续约价值证明（一页 A4）；(4) 中国本土化适配 -- 支持招投标流程节点管理、关键关系人维护提醒、财政/预算周期适配。

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| customer_id | string | 是 | 客户唯一标识 |
| renewal_date | date | 是 | 合同到期日 |
| pricing_strategy | enum | 否 | increase / flat / discount / auto，默认 auto |
| china_mode | boolean | 否 | 启用中国本土化模式（招标/关系），默认 true |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| war_room_dashboard | markdown | 续约作战仪表盘（倒计时 + 状态总览） |
| risk_assessment | object | 续约风险评估（red/yellow/green + 各维度评分） |
| pricing_recommendation | object | 定价建议（建议价格 + 理由 + 谈判区间） |
| value_one_pager | markdown | 价值一页纸（面向决策者） |
| timeline_plan | list | 倒计时行动计划（120d/90d/60d/30d/7d 节点） |
| stakeholder_strategy | list | 干系人攻略（谁需要搞定、如何搞定） |
| bidding_checklist | list | 招标准备清单（仅 china_mode=true 时输出） |

**数据源**: CRM API（合同到期日、金额 via MCP）, Health-Dashboard（健康分）, Value-Proof-Generator（价值数据）, Competitor-Radar（竞争情报）, Obsidian 客户文件

**降级策略**
- G1: 使用历史续约数据预估风险，注明"部分数据需更新"
- G2: 仅生成标准续约时间线 + 基础价值摘要
- G3: 返回续约准备 checklist + 信息收集清单

**协同关系**
- 依赖 <-: Health-Dashboard（健康数据）, Value-Proof-Generator（价值弹药）, Competitor-Radar（竞对情报）, Churn-Predict（流失风险）
- 触发 ->: Empathy-Writer（续约沟通邮件）, Executive-Briefing（高管汇报）, Workflow-Agent（续约审批流程）
- 被调用 <-: Report-Builder

**触发关键词**: 续约, 续费, 到期, renewal, 作战室, 续约风险, 合同到期, 续签

---

### Skill 11: 流失预测 (Churn-Predict)

| 字段 | 值 |
|------|-----|
| 适用阶段 | Adoption, Renewal |
| 自动化级别 | L2（AI 规则引擎预测，CSM 制定挽留方案） |
| 状态 | active |

**分层行为**
- **Strategic**: 多维流失信号分析 + 中国特色信号监测 + 相似案例 RAG 检索 + 挽留方案建议 + 根因分析；CSM 主导挽留
- **Growth**: 标准信号监测 + 风险评分 + Top 3 风险因素 + 相似案例参考；CSM 跟进高风险客户
- **Scale**: 关键指标阈值告警（健康分 < 40 或 DAU 连续下降）+ 自动触发关怀流程

**核心功能**
基于规则引擎（非机器学习）的流失风险预测系统。核心设计理念：在中国 B2B 市场，ML 模型的训练数据往往不足，规则引擎更透明可控。信号体系包括：(1) 通用信号 -- 健康分下降、使用量骤降、工单激增、NPS 低分、关键功能弃用；(2) 中国特色信号 -- 关键对接人离职/调岗（在中国 B2B 中极为常见且致命）、政策/预算周期变化（如政府客户年底预算调整）、组织架构调整、领导层更换。同时整合相似案例 RAG 检索，从历史流失/挽留案例中找到最相似的参考。

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| customer_id | string | 是 | 客户唯一标识 |
| signal_override | list | 否 | 手动追加的风险信号（如 CSM 观察到的定性信号） |
| include_similar_cases | boolean | 否 | 是否检索相似案例，默认 true |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| churn_risk_score | int | 流失风险评分（0-100，越高越危险） |
| risk_level | enum | red / yellow / green |
| risk_signals | list | 触发的风险信号列表（信号名 + 严重度 + 数据来源） |
| root_cause_analysis | markdown | 流失根因分析 |
| similar_cases | list | 相似历史案例（含结果：churned / saved + 挽留措施） |
| retention_playbook | markdown | 建议挽留方案 |
| china_specific_flags | list | 中国特色风险标记（对接人变动、政策周期等） |

**数据源**: Health-Dashboard（健康分）, Adoption-Analyzer（使用趋势）, Voice-of-Customer（情感数据）, Obsidian 客户文件（对接人信息）, 历史流失/挽留案例库

**降级策略**
- G1: 仅使用健康分趋势预测，跳过中国特色信号
- G2: 返回基础风险指标列表，无相似案例检索
- G3: 返回风险信号检查清单 + CSM 自评问卷

**协同关系**
- 依赖 <-: Health-Dashboard（健康趋势）, Adoption-Analyzer（使用数据）, Voice-of-Customer（情感信号）
- 触发 ->: Renewal-War-Room（高风险客户续约升级）, Empathy-Writer（挽留沟通）
- 被调用 <-: Executive-Briefing, Report-Builder

**触发关键词**: 流失, 流失风险, churn, 要丢了, 客户要走, 挽留, 风险预测, 对接人离职

---

### Skill 12: 共情写手 (Empathy-Writer)

| 字段 | 值 |
|------|-----|
| 适用阶段 | Handoff -> Renewal（全旅程通用） |
| 自动化级别 | L3（AI 自动生成 3 版本，CSM 选择/微调后发送） |
| 状态 | active |

**分层行为**
- **Strategic**: 高度定制化写作 + 深度背景融入 + 多风格版本 + 中国商务礼仪适配 + 关系维度考量；CSM 精细打磨
- **Growth**: 标准模板 + 情境定制 + 3 版本选择；CSM 快速选择微调
- **Scale**: 模板化生成 + 自动变量填充；CSM 一键发送

**核心功能**
面向中国 B2B 场景的专业沟通写手。遵循"安抚 -> 解释 -> 解决 -> 承诺"四步法（calm -> explain -> solve -> commit）生成高质量客户沟通文本。每次生成 3 个版本（正式/温和/简洁），CSM 选择最合适的版本后微调发送。深度适配中国商务礼仪：尊称使用、谦语表达、"先关系后事情"的沟通逻辑、节假日/特殊时期的语气调整。

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| scenario | enum | 是 | welcome / escalation_response / check_in / renewal_pitch / churn_save / expansion_intro / apology / congrats / holiday / custom |
| customer_id | string | 否 | 客户标识（用于上下文个性化） |
| context | string | 否 | 额外背景信息 |
| tone_preference | enum | 否 | formal / warm / concise / auto，默认 auto |
| channel | enum | 否 | email / wechat / dingtalk / letter，默认 email |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| versions | list | 3 个版本（formal / warm / concise），各含完整文本 |
| recommended_version | int | 推荐版本编号（1/2/3）+ 推荐理由 |
| four_step_breakdown | object | 四步法结构拆解（便于 CSM 理解写作逻辑） |
| etiquette_notes | list | 中国商务礼仪提示（如有特殊注意事项） |
| follow_up_timing | string | 建议跟进时间 |

**数据源**: Obsidian 客户文件（背景信息）, 沟通模板库, 中国商务礼仪规则库

**降级策略**
- G1: 使用通用模板 + 变量替换，跳过个性化
- G2: 仅生成 1 个版本（默认 warm 风格）
- G3: 返回沟通要点大纲 + 模板链接

**协同关系**
- 依赖 <-: KB-QA（产品知识支撑）, 各触发源 Skill 提供的上下文
- 触发 ->: 无（终端输出 Skill）
- 被调用 <-: Renewal-War-Room, Churn-Predict, Expansion-Radar, Sales-Handoff, 几乎所有需要客户沟通的 Skill

**触发关键词**: 写邮件, 写消息, 沟通, 回复客户, 道歉, 祝贺, 催续约, 欢迎邮件, empathy, 怎么说

---

### Skill 13: 报告构建器 (Report-Builder)

| 字段 | 值 |
|------|-----|
| 适用阶段 | Value Realization, Renewal |
| 自动化级别 | L3（AI 自动汇编生成，CSM 审核叙事） |
| 状态 | active |

**分层行为**
- **Strategic**: 定制化 QBR/EBR 报告 + 深度价值叙事 + 多格式输出（PPT/Word/MD）+ 高管定制版本；CSM 深度参与叙事设计
- **Growth**: 标准 QBR 模板 + 数据自动填充 + 价值摘要；CSM 审核后输出
- **Scale**: 简化季度摘要（1-2 页）+ 自动生成；无 CSM 深度参与

**核心功能**
自动化生成 QBR（Quarterly Business Review）和 EBR（Executive Business Review）报告。核心理念：报告的本质不是"展示数据"，而是"讲述价值故事"。以价值为中心的叙事结构：回顾目标 -> 展示成果 -> 量化价值 -> 规划未来。自动从各 Skill 汇编数据（健康分、使用数据、价值证明、VoC 摘要），生成可交付格式（PPT 推荐用于 Strategic 客户、Word/MD 用于内部存档）。

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| customer_id | string | 是 | 客户唯一标识 |
| report_type | enum | 是 | qbr / ebr / monthly_summary / custom |
| format | enum | 否 | pptx / docx / markdown，默认 markdown |
| time_range | string | 否 | 报告周期，如 90d / 180d，默认 90d |
| audience | enum | 否 | internal / customer_ops / customer_exec，默认 customer_ops |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| report_file | file | 生成的报告文件（PPT/Word/MD） |
| report_outline | markdown | 报告大纲（便于 CSM 快速预览） |
| data_sources_summary | list | 使用的数据源及其新鲜度 |
| narrative_highlights | list | 叙事亮点摘要（Top 3 故事线） |
| missing_data_flags | list | 缺失数据标记（可能影响报告质量的数据空白） |

**数据源**: Health-Dashboard, Adoption-Analyzer, Value-Proof-Generator, Voice-of-Customer, Expansion-Radar, TTV-Tracker -- 汇编型 Skill，依赖几乎所有分析型 Skill

**降级策略**
- G1: 仅填充可用数据，缺失部分标记为"[待补充]"
- G2: 生成纯 Markdown 格式，跳过 PPT/Word 排版
- G3: 返回报告框架大纲 + 数据收集清单

**协同关系**
- 依赖 <-: Health-Dashboard, Adoption-Analyzer, Value-Proof-Generator, Voice-of-Customer, Expansion-Radar, TTV-Tracker, Churn-Predict
- 触发 ->: Empathy-Writer（报告配套沟通邮件）
- 被调用 <-: Executive-Briefing（作为高管简报的数据底座）

**触发关键词**: QBR, EBR, 季度报告, 业务回顾, 报告, 汇报, 做个报告, 客户汇报

---

### Skill 14: 流程代理 (Workflow-Agent)

| 字段 | 值 |
|------|-----|
| 适用阶段 | Onboarding, Expansion, Renewal |
| 自动化级别 | L3（AI 自动构建表单，CSM 确认后提交） |
| 状态 | active |

**分层行为**
- **Strategic**: 智能流程编排 + 多系统联动 + 审批链路优化建议 + 流程状态追踪；CSM 监控关键节点
- **Growth**: 标准表单生成 + 单系统对接 + 状态通知；CSM 确认后自动提交
- **Scale**: 模板化表单 + 自动提交 + 异常通知

**核心功能**
将自然语言指令转换为 OA 系统可识别的 JSON 表单数据，对接钉钉（DingTalk）和企业微信（WeChat Work）的审批流程。CSM 用自然语言描述需求（如"帮客户 A 申请一个技术支持加急工单"），Workflow-Agent 自动识别流程类型、填充表单字段、生成 JSON 格式数据，经 CSM 确认后通过 API 提交。支持的流程类型：折扣审批、资源申请、工单升级、合同变更、客户投诉升级等。

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| instruction | string | 是 | 自然语言流程指令 |
| target_system | enum | 否 | dingtalk / wechat_work / auto，默认 auto |
| customer_id | string | 否 | 关联客户标识 |
| urgency | enum | 否 | normal / urgent / critical，默认 normal |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| form_json | object | 生成的 OA 表单 JSON 数据 |
| form_preview | markdown | 表单预览（人类可读格式） |
| workflow_type | string | 识别的流程类型 |
| approval_chain | list | 审批链路（审批人 + 预估时间） |
| submission_status | enum | draft / submitted / approved / rejected |
| tracking_id | string | 流程追踪编号 |

**数据源**: OA 系统表单模板配置, 审批链路规则, Obsidian 客户文件（关联信息）

**降级策略**
- G1: 生成表单 JSON 但不自动提交，由 CSM 手动提交
- G2: 生成表单预览文本，由 CSM 在 OA 系统中手动填写
- G3: 返回流程类型识别 + 所需填写字段清单

**协同关系**
- 依赖 <-: 各触发源 Skill 提供的流程需求上下文
- 触发 ->: 无（执行型 Skill）
- 被调用 <-: Renewal-War-Room（续约审批）, Expansion-Radar（增购审批）, Log-Analyzer（工单升级）

**触发关键词**: 审批, 流程, 提交, 申请, OA, 钉钉, 企业微信, 工单, 审批流

---

### Skill 15: 日志分析师 (Log-Analyzer)

| 字段 | 值 |
|------|-----|
| 适用阶段 | Onboarding, Adoption |
| 自动化级别 | L2（AI 分析诊断，CSM/技术支持审核确认） |
| 状态 | active |

**分层行为**
- **Strategic**: 深度日志分析 + 根因定位 + 解决方案推荐 + 历史工单关联 + KB 知识检索 + 升级建议；CSM 协调技术资源
- **Growth**: 标准日志分析 + 常见问题匹配 + KB 搜索结果；CSM 转发技术团队
- **Scale**: 错误关键词匹配 + FAQ 自动回复 + 升级链接

**核心功能**
分析客户提交的错误截图和日志文件，通过 OCR（截图）或文本解析（日志文件）提取关键错误信息，然后执行 RAG 检索：(1) 搜索历史工单库 -- 查找相似问题及解决方案；(2) 搜索知识库 -- 查找相关文档和最佳实践；(3) 搜索已知问题清单 -- 匹配已知 Bug 和 Workaround。整合结果生成诊断报告，帮助 CSM 快速响应客户技术问题，减少"我去问下技术同事"的等待时间。

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| input_type | enum | 是 | screenshot / log_text / log_file |
| content | string/file | 是 | 错误截图路径 / 日志文本 / 日志文件路径 |
| customer_id | string | 否 | 客户标识（用于关联客户环境配置） |
| product_version | string | 否 | 产品版本号 |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| diagnosis | markdown | 诊断报告（错误描述 + 可能原因 + 严重度） |
| matched_tickets | list | 匹配的历史工单（相似度 + 解决方案） |
| kb_results | list | 知识库匹配结果（文档 + 相关章节） |
| known_issues | list | 匹配的已知问题（Bug ID + 状态 + Workaround） |
| recommended_actions | list | 建议操作（按优先级排序） |
| escalation_needed | boolean | 是否需要升级至技术团队 |

**数据源**: 客户提交的截图/日志, 历史工单库, KB-QA（知识库）, 已知问题清单（Obsidian）

**降级策略**
- G1: 仅做关键词匹配，跳过语义分析
- G2: 返回原始错误信息 + 通用排查步骤清单
- G3: 直接创建工单升级至技术团队

**协同关系**
- 依赖 <-: KB-QA（知识库检索）
- 触发 ->: Workflow-Agent（工单升级流程）, Health-Dashboard（support_tickets 维度更新）
- 被调用 <-: 无（入口级 Skill，由客户问题触发）

**触发关键词**: 报错, 错误, 日志, log, error, 截图, 出问题了, bug, 异常, 崩溃

---

### Skill 16: 场景演练教练 (Role-Play-Coach)

| 字段 | 值 |
|------|-----|
| 适用阶段 | Adoption, Renewal |
| 自动化级别 | L2（AI 模拟客户角色，CSM 练习应对） |
| 状态 | active |

**分层行为**
- **Strategic**: 深度角色模拟 + 多轮对话 + 实时评分 + 话术优化建议 + 复盘分析 + 定制客户画像；CSM 主动使用提升技能
- **Growth**: 标准角色模拟 + 评分 + 改进建议；CSM 按需练习
- **Scale**: 简化问答练习 + 标准话术参考；自学模式

**核心功能**
模拟真实客户角色，帮助 CSM 在安全环境中练习高难度对话。预设 3 类典型客户角色：(1) 价格敏感型（Price-Sensitive）-- 反复纠缠价格、要求折扣、比较竞对价格，训练 CSM 价值引导能力；(2) 技术怀疑型（Tech-Skeptic）-- 质疑产品能力、提出刁钻技术问题、要求 POC，训练 CSM 技术应对能力；(3) 情绪激动型（Emotional）-- 因故障/体验差而愤怒、威胁取消合同、要求赔偿，训练 CSM 情绪管理和危机应对能力。支持多轮对话，每轮结束后给出评分和改进建议。

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| persona | enum | 是 | price_sensitive / tech_skeptic / emotional / custom |
| scenario | enum | 否 | renewal_negotiation / escalation_handling / expansion_pitch / onboarding_kickoff / custom |
| difficulty | enum | 否 | easy / medium / hard，默认 medium |
| custom_persona_desc | string | 否 | 自定义客户角色描述（persona=custom 时必填） |
| customer_id | string | 否 | 基于真实客户画像模拟（可选） |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| conversation_log | list | 对话记录 |
| score_card | object | 评分卡（沟通技巧、专业知识、情绪管理、价值传递、整体表现，各 0-100） |
| improvement_tips | list | 改进建议（具体到某句话应该怎么说） |
| best_practice_reference | markdown | 对应场景的最佳实践话术参考 |
| replay_highlights | list | 复盘亮点和失误标记 |

**数据源**: 客户角色模板库, 场景剧本库, KB-QA（产品知识支撑角色提问）, Competitor-Radar（竞对知识支撑角色提问）

**降级策略**
- G1: 使用固定剧本替代动态角色，减少灵活性但保证体验
- G2: 仅提供话术参考手册，无交互式模拟
- G3: 返回场景描述 + 推荐应对策略文档

**协同关系**
- 依赖 <-: KB-QA（产品知识支撑）, Competitor-Radar（竞对信息支撑）
- 触发 ->: 无（训练型 Skill，不直接影响客户）
- 被调用 <-: 无（CSM 主动调用）

**触发关键词**: 演练, 模拟, 练习, 角色扮演, role play, 怎么应对, 教练, 难缠客户, 怎么回

---

### Skill 17: 高管简报 (Executive-Briefing)

| 字段 | 值 |
|------|-----|
| 适用阶段 | Value Realization -> Renewal（仅 Strategic 层级客户） |
| 自动化级别 | L3（AI 自动生成，CSM 精细打磨） |
| 状态 | active |

**分层行为**
- **Strategic**: 完整高管简报包 + 一页纸摘要 + talking points + ask 清单 + 风险预案；CSM 与管理层共同打磨
- **Growth**: 不适用（Growth 层级客户通常不需要高管简报）
- **Scale**: 不适用

**核心功能**
专为 Strategic 层级客户设计的高管简报工具，面向 VP/SVP/C-level 决策者。核心产出：(1) 一页纸摘要（Executive One-Pager）-- 在一页 A4 内呈现客户全景：健康度、价值实现、风险、机会、下一步，字号 >= 12pt，关键数字加粗；(2) Talking Points -- 3-5 条结构化谈话要点，每条包含"要说什么 + 为什么说 + 预期反应 + 应对方案"；(3) Ask 清单 -- 本次会面要向客户高管争取的具体承诺/资源/决策，按优先级排列。设计哲学："高管的时间是最贵的资源，每一句话都要有目的。"

**输入 (Inputs)**

| 参数 | 类型 | 必填 | 说明 |
|------|------|:----:|------|
| customer_id | string | 是 | 客户唯一标识 |
| meeting_type | enum | 是 | qbr_exec / ebr / escalation / renewal / relationship / custom |
| attendees | list | 否 | 参会高管列表（姓名 + 职位 + 关注点） |
| our_asks | list | 否 | 我方诉求/请求清单 |
| time_limit_minutes | int | 否 | 会议时长（用于控制内容量），默认 30 |

**输出 (Outputs)**

| 字段 | 类型 | 说明 |
|------|------|------|
| executive_one_pager | markdown | 一页纸高管摘要 |
| talking_points | list | 谈话要点（每条含 what/why/expected_reaction/response） |
| ask_list | list | Ask 清单（优先级排序 + 备选方案） |
| risk_contingency | list | 风险预案（可能的尴尬问题 + 应对） |
| briefing_deck | markdown | 完整简报文档（可导出 PPT） |
| pre_meeting_checklist | list | 会前准备清单 |

**数据源**: Health-Dashboard（全景数据）, Value-Proof-Generator（价值数据）, Report-Builder（历史报告）, Competitor-Radar（竞争态势）, Churn-Predict（风险评估）, Obsidian 客户文件（干系人信息）

**降级策略**
- G1: 使用历史数据 + 标准模板生成，注明"部分数据需更新"
- G2: 仅生成 talking points + ask 清单，跳过 one-pager
- G3: 返回简报框架 + 数据收集清单

**协同关系**
- 依赖 <-: Health-Dashboard, Value-Proof-Generator, Report-Builder, Competitor-Radar, Churn-Predict, Voice-of-Customer
- 触发 ->: Empathy-Writer（会后跟进邮件）, Workflow-Agent（会议承诺转 Action Item）
- 被调用 <-: 无（CSM/管理层主动调用）

**触发关键词**: 高管简报, executive briefing, 见 VP, 见高管, 一页纸, talking points, 高层会面, 战略客户汇报

---

## 三、Skill 间协同总览

```
Sales-Handoff ──→ TTV-Tracker ──→ Adoption-Analyzer ──→ Value-Proof-Generator
      │                │                  │                       │
      ▼                ▼                  ▼                       ▼
Health-Dashboard ←──── * ────── * ──────── * ─────────── * ──→ Renewal-War-Room
      │                                                           │
      ▼                                                           ▼
Churn-Predict ─────────────────────────────────────────────→ Empathy-Writer
      │
      ▼
Expansion-Radar ──→ Competitor-Radar

KB-QA ←── 被所有 Skill 调用（基础设施级）
Report-Builder ←── 汇编所有分析型 Skill 输出
Executive-Briefing ←── 汇编最高层级摘要
Workflow-Agent ←── 执行所有流程类操作
Log-Analyzer ←── 技术问题入口
Role-Play-Coach ←── CSM 训练工具（独立）
Voice-of-Customer ──→ 反馈信号流入 Health-Dashboard + Churn-Predict
```

---

## 四、自动化级别定义参考

| 级别 | 名称 | 描述 |
|------|------|------|
| L1 | 建议 | AI 仅提供分析和建议，CSM 决策并执行 |
| L2 | 起草 | AI 生成草稿供审阅，CSM 修改后执行 |
| L3 | 审批执行 | AI 准备材料，CSM 审批后 AI 执行 |
| L4 | 半自动 | 低风险操作自动执行，高风险操作等待审批 |
| L5 | 全自动 | AI 自动执行并报告，CSM 仅监督 |

> 与 SKILL.md 「安全与自动化治理」章节保持一致。

---

## 五、数据类型约束规范

本文档中所有 Skill 输入/输出参数的数据类型统一约束如下：

| 参数类型 | 声明 | 格式约束 | 示例 |
|---------|------|---------|------|
| `time_range` | string | 支持 `7d` / `30d` / `90d` / `180d` / `1y` 标准格式；也支持日期区间如 `2026-01-01~2026-03-31` | `30d` |
| `customer_id` | string | 大小写敏感，不支持空格，建议驼峰或下划线命名 | `customer_001` |
| `date` | string | ISO-8601 日期格式 `YYYY-MM-DD` | `2026-04-23` |
| `enum` | string | 仅限定义值列表中的取值，大小写敏感 | `standard` / `deep` |
| `list` | list[str] | 逗号分隔字符串或 JSON 数组格式 | `["usage", "behavior"]` |
| `float` | float | 0.0 - 1.0 范围（置信度等），保留两位小数 | `0.85` |
| `int` | int | 非负整数，部分参数有上限（如置信度阈值 0-100） | `75` |
| `boolean` | boolean | `true` / `false` 或 `yes` / `no` | `true` |
| `markdown` | string | 标准 Markdown 格式，支持表格/列表/代码块 | 见各输出示例 |

**校验规则**：
1. 参数传入时先校验类型和格式，不符合时返回 `400 Bad Request` 等效提示并告知正确格式。
2. `time_range` 传入无法解析的值时，回退到默认值 `30d` 并标注 `[time_range 格式未识别，使用默认 30d]`。
3. 日期参数不支持相对描述（如"上周""本月"），必须转换为标准格式后传入。

---

> 文档版本: v1.0
> 最后更新: 2026-04-23
> 维护者: SkyHub Core Team
