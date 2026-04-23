# Obsidian知识库结构

> SkyHub的所有数据持久化于本地Obsidian Vault，零云端依赖。本文档定义Vault的目录结构、文件格式、命名规范、Frontmatter模板及Dataview查询模式。

---

## Vault目录结构

```
SkyHub-KB/
├── 00-Dashboards/           # 仪表盘
│   ├── risk-alerts.md
│   ├── my-customer-pool.md
│   ├── team-ops-cockpit.md
│   ├── renewal-calendar.md
│   └── tech-touch-weekly.md
├── 01-Customers/            # 客户档案
│   ├── Strategic/
│   ├── Growth/
│   └── Scale/
├── 02-Playbooks/            # Playbook库
│   ├── lifecycle/           # 按生命周期阶段
│   ├── cross-stage/         # 跨阶段通用
│   ├── scale/               # Scale专属自动化
│   ├── executions/          # 执行记录
│   └── _templates/          # Playbook模板
├── 03-Skills/               # 17个Skill文件夹
│   ├── S01-HealthPulse/
│   ├── S02-OnboardingAccel/
│   ├── ...
│   └── S17-StratAlign/
├── 04-Knowledge/            # 通用知识库
│   ├── Product_Docs/
│   ├── FAQ/
│   ├── Competitor_Intel/
│   ├── Industry/
│   │   ├── finance/
│   │   ├── manufacturing/
│   │   └── internet/
│   ├── Case_Library/
│   └── Best_Practices/
├── 05-VoC/                  # 客户之声
│   ├── NPS/
│   ├── CSAT/
│   └── Feedback_Log/
├── 06-Reports/              # 生成的报告
│   ├── QBR/
│   ├── EBR/
│   ├── Value_Reports/
│   └── Health_Reports/
├── 07-CSM_Profiles/         # CSM画像
├── 08-Learnings/            # 经验库
│   ├── churn-postmortems/
│   ├── success-stories/
│   └── best-practices/
├── _config/                 # 系统配置
│   ├── health_score_rules.yaml
│   ├── health_score_versions/
│   ├── data_masking_rules.yaml
│   ├── automation_levels.yaml
│   └── tier_thresholds.yaml
└── _governance/             # 治理与审计
    ├── automation_level_changelog.md
    ├── degradation_log.md
    ├── risk_dashboard.md
    └── skill_audit_log.md
```

---

## 各目录详细说明

### 00-Dashboards/
- **用途**: 汇聚全局视图，CSM每日打开的第一个入口。通过Dataview查询动态聚合客户、风险、续约等信息。
- **文件格式**: Markdown + 内嵌Dataview查询块。
- **命名规范**: `kebab-case.md`，如 `risk-alerts.md`。
- **示例文件**: `risk-alerts.md`, `my-customer-pool.md`, `team-ops-cockpit.md`, `renewal-calendar.md`, `tech-touch-weekly.md`。
- **Dataview能力**: 所有仪表盘均通过 `dataview` 或 `dataviewjs` 实时查询 `01-Customers/` 下的Frontmatter字段，无需手动更新。

### 01-Customers/
- **用途**: 每个客户一个Markdown文件，承载全量客户档案。按Tier分子目录便于批量操作。
- **文件格式**: Markdown，强制Frontmatter（见下文模板）。正文包含客户纪要、关键事件时间线。
- **命名规范**: `{customer_id}-{customer_name}.md`，如 `C001-字节跳动.md`。
- **示例文件**: `Strategic/C001-字节跳动.md`, `Growth/C042-瑞幸咖啡.md`, `Scale/C200-小微科技.md`。
- **Dataview能力**: 通过 `tier`, `health_level`, `lifecycle_stage`, `contract_end` 等字段可实现任意维度的筛选、排序、分组。

### 02-Playbooks/
- **用途**: 存储Playbook定义文件与执行记录。定义文件描述流程步骤，执行记录绑定到具体客户。
- **文件格式**: Playbook定义为YAML Frontmatter + Markdown步骤描述；执行记录为独立Markdown文件。
- **命名规范**: 定义文件 `PB-{id}-{name}.md`；执行记录 `EXEC-{date}-{customer_id}-{pb_id}.md`。
- **示例文件**: `lifecycle/PB-L01-onboarding-kickoff.md`, `executions/EXEC-20260415-C001-L01.md`。
- **Dataview能力**: 可查询"某客户正在执行的Playbook"、"某Playbook的历史执行成功率"等。

### 03-Skills/
- **用途**: 每个Skill一个子目录，包含Skill描述、脚本、模板、示例输出。
- **文件格式**: `SKILL.md` 为入口，脚本为 `.py` / `.sh`，模板为 `.md`。
- **命名规范**: `S{nn}-{SkillName}/`，如 `S01-HealthPulse/`。
- **示例文件**: `S01-HealthPulse/SKILL.md`, `S01-HealthPulse/scripts/calc_health.py`。
- **Dataview能力**: Skill元信息通过Frontmatter暴露，可查询"哪些Skill适用于Scale Tier"等。

### 04-Knowledge/
- **用途**: 产品文档、FAQ、竞品情报、行业知识、案例库、最佳实践。CSM和AI共同消费的知识底座。
- **文件格式**: Markdown，Frontmatter含 `category`, `tags`, `updated`, `source`。
- **命名规范**: 按子目录语义自由命名，建议 `kebab-case`。
- **示例文件**: `Product_Docs/feature-api-gateway.md`, `Competitor_Intel/competitor-A-battlecard.md`, `Industry/finance/banking-cs-trends-2026.md`。
- **Dataview能力**: 通过 `tags` 和 `category` 实现跨知识域的关联搜索，支持Skill在运行时自动引用相关知识。

### 05-VoC/
- **用途**: 客户之声（Voice of Customer），存储NPS评分、CSAT记录、反馈日志，驱动S05-VoCInsight分析。
- **文件格式**: Markdown或CSV（CSV通过mcp-csv-reader解析）。
- **命名规范**: `{type}-{date}-{customer_id}.md`，如 `NPS-20260401-C001.md`。
- **示例文件**: `NPS/NPS-20260401-C001.md`, `Feedback_Log/feedback-20260420-C042.md`。
- **Dataview能力**: 可聚合NPS趋势、CSAT分布，按客户/时间段/产品线切片分析。

### 06-Reports/
- **用途**: Skill或CSM生成的正式报告，包括QBR、EBR、价值报告、健康报告。只读归档。
- **文件格式**: Markdown（可导出为PDF/PPTX）。
- **命名规范**: `{type}-{date}-{customer_id}.md`，如 `QBR-2026Q1-C001.md`。
- **示例文件**: `QBR/QBR-2026Q1-C001.md`, `Value_Reports/value-20260415-C042.md`。
- **Dataview能力**: 可查询"某客户的所有历史报告"、"本季度待生成QBR的客户列表"。

### 07-CSM_Profiles/
- **用途**: CSM个人画像，包含负责客户列表、技能标签、绩效指标、成长路径。
- **文件格式**: Markdown + Frontmatter。
- **命名规范**: `CSM-{name}.md`，如 `CSM-张三.md`。
- **示例文件**: `CSM-张三.md`, `CSM-李四.md`。
- **Dataview能力**: 可查询"每个CSM负责的客户数/ARR总额"、"Skill使用频次排行"。

### 08-Learnings/
- **用途**: 流失复盘、成功案例、最佳实践。组织级知识沉淀，驱动持续改进。
- **文件格式**: Markdown + Frontmatter（含 `type`, `customer_id`, `root_cause`, `lessons`）。
- **命名规范**: `{type}-{date}-{customer_id}.md`。
- **示例文件**: `churn-postmortems/churn-20260301-C099.md`, `success-stories/success-20260401-C001.md`。
- **Dataview能力**: 可按流失原因分类聚合、按行业筛选成功案例，用于S16-LearningLoop自动推荐。

### _config/
- **用途**: 系统级配置文件，控制健康分规则、数据脱敏规则、自动化等级阈值、Tier划分标准。
- **文件格式**: YAML。
- **命名规范**: `{config_name}.yaml`，版本化文件存入 `health_score_versions/`。
- **示例文件**: `health_score_rules.yaml`, `tier_thresholds.yaml`, `health_score_versions/v2.1.yaml`。
- **Dataview能力**: 配置文件本身不参与Dataview查询，但其内容被Skill脚本在运行时读取。

### _governance/
- **用途**: 治理与审计日志，记录自动化等级变更、系统降级事件、Skill审计轨迹。
- **文件格式**: Markdown日志。
- **命名规范**: 固定文件名，追加写入。
- **示例文件**: `automation_level_changelog.md`, `degradation_log.md`, `skill_audit_log.md`。
- **Dataview能力**: 可通过日志中的结构化标记（如日期、事件类型）进行过滤查询。

---

## 客户档案Frontmatter模板

```yaml
---
customer_id: "C001"
customer_name: "字节跳动"
industry: "互联网"
company_size: "10000+"
tier: "Strategic"          # Strategic | Growth | Scale
client_type: "commercial"     # commercial | soe | government | mnc
payment_terms_days: 30         # 约定付款周期（天）
fiscal_year_start: "01-01"     # 财年起始月日
csm: "张三"
csm_manager: "李四"
arr: 1200000               # 年度经常性收入（元）
contract_start: 2025-01-15
contract_end: 2026-01-14
products:
  - "API网关"
  - "数据分析平台"
  - "安全审计"
contracts:
  - contract_id: "CTR-001"
    product: "API网关"
    arr: 800000
    start_date: 2025-01-15
    end_date: 2026-01-14
    lifecycle_stage: "expanding"
    renewal_status: "on_track"
  - contract_id: "CTR-002"
    product: "数据分析平台"
    arr: 400000
    start_date: 2025-06-01
    end_date: 2026-05-31
    lifecycle_stage: "adopting"
    renewal_status: "not_started"
payment_status: "正常"      # 正常 | 逾期 | 催收中
lifecycle_stage: "expanding" # onboarding | adopting | expanding | renewing | churning
onboarding_completed: true
ttv_days: 23                # Time to Value（天）
first_value_milestone: "首次API调用量突破100万/日"

contacts:
  - name: "王总"
    role: "VP Engineering"
    type: "executive_sponsor"
    relationship_temp: "warm"   # hot | warm | cold
  - name: "赵工"
    role: "技术负责人"
    type: "champion"
    relationship_temp: "hot"
  - name: "钱经理"
    role: "采购经理"
    type: "economic_buyer"
    relationship_temp: "warm"

success_criteria:
  - metric: "API可用性"
    target: "99.95%"
    current: "99.97%"
    status: "on_track"       # on_track | at_risk | off_track
  - metric: "平均响应时间"
    target: "<50ms"
    current: "42ms"
    status: "on_track"

health_score: 82
health_level: "Green"          # Green | Yellow | Red
health_model_version: "v2.1"
health_updated: 2026-04-20
health_dimensions:
  usage_depth: 85
  activity_trend: 78
  commercial_health: 90
  support_tickets: 80
  stakeholder_engagement: 75
  value_realization: 88
  sentiment: 70
health_trend: "stable"       # improving | stable | declining
health_30d_delta: -2

risk_flags:
  - "champion_left:2026-03-15"
  - "support_ticket_spike:2026-04-10"

active_playbooks:
  - "PB-L04-expansion-discovery"
  - "PB-X02-champion-change"

tags:
  - "互联网"
  - "大客户"
  - "API网关"
  - "续约预警"
---
```

---

## 仪表盘Dataview查询模板

### 风险预警仪表盘 (risk-alerts.md)

```dataview
TABLE
  customer_name AS "客户",
  tier AS "等级",
  health_score AS "健康分",
  health_trend AS "趋势",
  risk_flags AS "风险标记",
  contract_end AS "合同到期"
FROM "01-Customers"
WHERE health_level = "Red"
SORT health_score ASC
```

### 我的客户池 (my-customer-pool.md)

```dataview
TABLE
  customer_name AS "客户",
  tier AS "等级",
  arr AS "ARR",
  lifecycle_stage AS "阶段",
  health_score AS "健康分",
  health_level AS "健康状态"
FROM "01-Customers"
WHERE csm = "张三"
SORT tier ASC, health_score ASC
```

### 团队运营驾驶舱 (team-ops-cockpit.md)

```dataviewjs
const customers = dv.pages('"01-Customers"');
const totalARR = customers.values.reduce((sum, c) => sum + (c.arr || 0), 0);
const avgHealth = customers.values.reduce((sum, c) => sum + (c.health_score || 0), 0) / customers.length;
const atRisk = customers.where(c => c.health_level === "Red").length;

dv.header(3, "团队概览");
dv.paragraph(`客户总数: ${customers.length} | 总ARR: ¥${(totalARR/10000).toFixed(0)}万 | 平均健康分: ${avgHealth.toFixed(1)} | 风险客户: ${atRisk}`);
```

### 续约日历 (renewal-calendar.md)

```dataview
TABLE
  customer_name AS "客户",
  tier AS "等级",
  arr AS "ARR",
  contract_end AS "到期日",
  health_score AS "健康分",
  health_level AS "健康状态"
FROM "01-Customers"
WHERE contract_end <= date(today) + dur(90 days)
SORT contract_end ASC
```

### Tech-touch周报 (tech-touch-weekly.md)

```dataview
TABLE
  customer_name AS "客户",
  health_score AS "健康分",
  health_30d_delta AS "30天变化",
  active_playbooks AS "活跃Playbook"
FROM "01-Customers"
WHERE tier = "Scale"
SORT health_30d_delta ASC
LIMIT 20
```

---

## 双向链接模式

### Wikilink使用规范

- 客户档案引用Playbook: `当前正在执行 [[PB-L04-expansion-discovery]]`
- Playbook执行记录引用客户: `客户: [[C001-字节跳动]]`
- Skill输出引用知识: `参考 [[feature-api-gateway]] 的最新更新`
- 经验库引用客户: `流失客户 [[C099-某科技]] 的复盘分析`

### Tag体系

| Tag前缀 | 用途 | 示例 |
|---------|------|------|
| `#tier/` | 客户等级 | `#tier/strategic` |
| `#stage/` | 生命周期 | `#stage/onboarding` |
| `#risk/` | 风险类型 | `#risk/champion-left` |
| `#industry/` | 行业 | `#industry/finance` |
| `#skill/` | 关联Skill | `#skill/S01-HealthPulse` |
| `#playbook/` | 关联Playbook | `#playbook/PB-L01` |

### 知识图谱构建

通过双向链接和标签，Obsidian自动形成CS知识图谱：
- **客户节点**: 连接到其Playbook执行记录、健康报告、VoC数据。
- **Playbook节点**: 连接到所有执行过该Playbook的客户、关联的Skill。
- **知识节点**: 被多个Skill和Playbook引用，形成知识复用网络。

---

## Graph View使用

### 可视化客户-Skill-Playbook关系

1. **全局视图**: 打开Graph View，可看到以客户为核心的星形拓扑。Strategic客户节点最大（因链接最多）。
2. **过滤视图**: 使用Graph View的过滤器，仅显示 `01-Customers/` 和 `02-Playbooks/` 目录，观察Playbook覆盖情况。
3. **Skill关联视图**: 过滤 `03-Skills/` 和 `04-Knowledge/`，观察Skill对知识库的依赖关系，识别知识盲区。
4. **风险传播视图**: 以某个风险客户为起点，沿链接追溯其Playbook执行历史、相关经验库条目，辅助根因分析。
5. **颜色编码**: 建议按目录设置不同颜色 — 客户(蓝)、Playbook(绿)、Skill(橙)、知识(灰)、报告(紫)。

### 局部图谱（Local Graph）

在任意客户档案页面打开Local Graph，可快速查看：
- 该客户关联的所有Playbook执行记录
- 引用该客户的报告和经验库条目
- 该客户涉及的知识文档

这是CSM快速理解单一客户全貌的最高效方式。
