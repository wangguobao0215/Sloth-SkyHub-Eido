# 深信 · 客户成功 — 静态功能测试报告

> 版本：v1.0.0 | 测试日期：2026-04-24 | 测试类型：基于文档的静态审查

---

## 一、测试范围

基于以下文档进行功能点审查与测试用例设计：
- SKILL.md（技能定义与路由）
- README.md（功能说明）
- references/*.md（领域知识参考）
- user-guide.md（用户使用指南）

---

## 二、文档完整性检查

| 检查项 | 状态 | 备注 |
|--------|------|------|
| SKILL.md 存在且结构完整 | ✅ | 包含角色定义、17 个子 Skill 路由、健康评分、Playbook 引擎、客户旅程、客户分层、安全治理，共 26.8 KB |
| README.md 存在且与 SKILL.md 版本一致 | ✅ | 版本号均为 v1.0.0，功能描述一致 |
| references/ 目录存在 | ✅ | 包含 13 个参考文件：skill-matrix.md、health-score-model.md、playbook-engine.md、customer-journey.md、tier-strategy.md、metrics-kpi.md、security-governance.md、obsidian-vault.md、mcp-integration.md、implementation-roadmap.md、risk-matrix.md、skill-integration.md、user-guide.md |
| user-guide.md 存在 | ✅ | 11.6 KB，覆盖安装、子 Skill 路由表、工作流程、关键系统说明、最佳实践、FAQ |
| 关键 references 文件齐全 | ✅ | 客户旅程、分层策略、Skill 规格、健康评分、Playbook 引擎、度量体系、安全治理、知识库结构、MCP 集成、实施路线图、风险矩阵、Skill 协同均存在 |

---

## 三、功能模块测试用例

### 模块 1：首次使用引导与知识库初始化

| 用例ID | 测试场景 | 输入/触发条件 | 预期输出 | 测试状态 |
|--------|----------|---------------|----------|----------|
| TC-101 | 首次使用自动引导 | 用户第一次触发 SkyHub | 执行 5 步引导：欢迎介绍 → 数据导入确认 → 知识库初始化检查 → 推荐起步路径 → 解释 4 期采纳旅程 | ✅ 文档描述完整 |
| TC-102 | Obsidian Vault 初始化 | Vault 目录不存在 | 自动创建标准目录结构：customers/、playbooks/、templates/、health-snapshots/、meeting-notes/ | ✅ 文档描述完整 |
| TC-103 | 数据导入引导 | 用户有现成客户数据（Excel/CSV/CRM 导出） | 引导上传并解析；无数据则跳过，后续手动录入 | ✅ 文档描述完整 |
| TC-104 | 起步路径推荐 | 初始化完成后 | 推荐从 KB-QA 和 Health-Dashboard 两个 Skill 开始 | ✅ 文档描述完整 |
| TC-105 | 4 期实施路线说明 | 首次引导最后一步 | 简要说明 P0 单点突破 → P1 流程串联 → P2 数据驱动 → P3 全面运营 | ✅ 文档描述完整 |

### 模块 2：意图路由器（17 个子 Skill + 元功能路由）

| 用例ID | 测试场景 | 输入/触发条件 | 预期输出 | 测试状态 |
|--------|----------|---------------|----------|----------|
| TC-201 | 精确关键词匹配 - Sales-Handoff | 用户输入"新客户交接" | 匹配 Sales-Handoff Skill，适用阶段 Onboarding，自动化上限 L2 | ✅ 文档描述完整 |
| TC-202 | 精确关键词匹配 - Health-Dashboard | 用户输入"看一下 XX 客户的健康分" | 匹配 Health-Dashboard Skill，适用全阶段，自动化上限 L4 | ✅ 文档描述完整 |
| TC-203 | 精确关键词匹配 - Renewal-War-Room | 用户输入"续约作战室" | 匹配 Renewal-War-Room Skill，适用阶段 Renewal，自动化上限 L3 | ✅ 文档描述完整 |
| TC-204 | 元功能路由 - Playbook 引擎 | 用户输入"启动新客户 Onboarding Playbook" | 路由到 Playbook Engine，启动/查看/管理 Playbook 实例 | ✅ 文档描述完整 |
| TC-205 | 元功能路由 - 客户切换 | 用户输入"切换到 XX 客户" | 更新当前 customer_id，加载该客户上下文 | ✅ 文档描述完整 |
| TC-206 | 复合意图分解 | 用户输入"帮我看一下 XX 客户的健康分和续约风险" | 分解为 DAG：Health-Dashboard → Churn-Predict，告知用户执行计划并确认 | ✅ 文档描述完整 |
| TC-207 | Fallback 能力菜单 | 无法识别意图时 | 展示 6 大能力分类菜单：客户健康全景、上线与采纳、价值证明、续约与扩展、沟通与报告、知识与分析 | ✅ 文档描述完整 |

### 模块 3：客户旅程六阶段与 Skill 激活

| 用例ID | 测试场景 | 输入/触发条件 | 预期输出 | 测试状态 |
|--------|----------|---------------|----------|----------|
| TC-301 | 阶段一：销售交接（Handoff） | 新签合同进入系统 | 激活 Sales-Handoff、KB-QA、Empathy-Writer；目标：完成交接、建立信任、创建 30-60-90 天计划 | ✅ 文档描述完整 |
| TC-302 | 阶段二：启用上线（Onboarding） | Day 7-90 | 激活 TTV-Tracker、KB-QA、Health-Dashboard、Empathy-Writer、Log-Analyzer；目标：最短 TTV | ✅ 文档描述完整 |
| TC-303 | 阶段三：深度采纳（Adoption） | Day 90-270 | 激活 Adoption-Analyzer、Expansion-Radar、Churn-Predict、Role-Play-Coach 等；目标：功能渗透、用户扩展 | ✅ 文档描述完整 |
| TC-304 | 阶段流转 - Handoff → Onboarding | 交接会议完成 + 客户档案初始化 + Success Plan 确认 | 自动流转至 Onboarding 阶段 | ✅ 文档描述完整 |
| TC-305 | 阶段流转 - Onboarding → Adoption | 首个价值里程碑达成 + Onboarding 正式关闭 | 自动流转至 Adoption 阶段 | ✅ 文档描述完整 |
| TC-306 | 并行分支 - Expansion | 检测到扩展信号（usage_depth ≥80 且 activity_trend ≥75） | 可从 Adoption / Value Realization / Renewal 任意阶段触发，进入扩展培育流程 | ✅ 文档描述完整 |
| TC-307 | 阶段七：客户代言（Advocacy） | 健康分连续 2 季度 Green + NPS ≥9 + 至少 1 次成功续约 | 激活 Voice-of-Customer、Empathy-Writer、Executive-Briefing、Value-Proof-Generator | ✅ 文档描述完整 |

### 模块 4：三层客户分级与分层行为

| 用例ID | 测试场景 | 输入/触发条件 | 预期输出 | 测试状态 |
|--------|----------|---------------|----------|----------|
| TC-401 | 战略客户（Strategic）自动判定 | ARR Top 20%，决策链长，定制需求多 | AI 角色 = 幕后军师；High-touch 模式：AI 准备材料，CSM 亲自交付 | ✅ 文档描述完整 |
| TC-402 | 成长客户（Growth）自动判定 | ARR 中段，标准需求为主 | AI 角色 = 副驾驶；Mid-touch 模式：AI 起草初稿+建议，CSM 审阅后执行 | ✅ 文档描述完整 |
| TC-403 | 规模客户（Scale）自动判定 | ARR 长尾，标准化场景 | AI 角色 = 主角；Tech-touch 模式：AI 自动执行低风险操作（L4-L5） | ✅ 文档描述完整 |
| TC-404 | 分层手动覆盖 | CSM 手动调整客户层级 | 系统允许手动覆盖自动分层结果 | ✅ 文档描述完整 |
| TC-405 | 跨层迁移触发 | Scale 客户 ARR 增长超过 Growth 阈值 | 自动提醒 CS Ops 评估层级升级 | ✅ 文档描述完整 |

### 模块 5：健康评分系统（7 维模型）

| 用例ID | 测试场景 | 输入/触发条件 | 预期输出 | 测试状态 |
|--------|----------|---------------|----------|----------|
| TC-501 | 七维度综合评分计算 | 各维度得分 × 默认权重 | 总分 = usage_depth×25% + activity_trend×20% + commercial_health×15% + support_tickets×10% + stakeholder_engagement×15% + value_realization×10% + sentiment×5% | ✅ 文档描述完整 |
| TC-502 | 阶段动态权重调整 | 客户处于 Onboarding 阶段 | usage_depth 权重上调至 20%，activity_trend 上调至 30%，commercial_health 下调至 5% | ✅ 文档描述完整 |
| TC-503 | 健康等级判定 | 总分计算完成 | Green ≥70 / Yellow 40-69 / Red <40 | ✅ 文档描述完整 |
| TC-504 | 降级处理 - 数据缺失 | 某维度数据源不可用 | 标注 `[数据缺失]`，不参与加权计算，总权重重新归一化 | ✅ 文档描述完整 |
| TC-505 | 季度校准流程 | 每季度末自动触发 | 执行数据回测 → CSM 反馈修正 → 校准会议 → 版本管理（存储于 `_config/health_score_versions/`） | ✅ 文档描述完整 |

### 模块 6：Playbook 引擎（10 个核心剧本）

| 用例ID | 测试场景 | 输入/触发条件 | 预期输出 | 测试状态 |
|--------|----------|---------------|----------|----------|
| TC-601 | PB-HND-001 新客户标准 Onboarding | contract_signed = true AND day_since_signing ≤ 1 | 执行 8 步骤序列：内部移交 → 欢迎邮件 → Kickoff → 技术部署 → 培训 → 首次价值检查 → Onboarding 总结 | ✅ 文档描述完整 |
| TC-602 | PB-RNW-001 续约倒计时 120 天 | 合同到期前 120 天自动触发 | 执行 8 步骤：续约评估 → 内部对齐 → 意向沟通 → 价值回顾 → 正式提案 → 谈判推进 → 签约冲刺 → 完成/流失记录 | ✅ 文档描述完整 |
| TC-603 | PB-ALL-001 客户健康分骤降响应 | health_score 7 天内下降 ≥15 分 | 14 天紧急响应：自动告警 → 根因定位 → 干预计划 → 客户沟通 → 密集执行 → 中期评估 → 闭环复盘 | ✅ 文档描述完整 |
| TC-604 | Playbook 并发优先级 | 同一客户同时触发多个 Playbook | 按 P0-P5 优先级执行：高风险挽留(P0) > 健康骤降/关键人变动(P1) > 续约倒计时(P2) > 扩展培育/QBR(P3) > 常规运营(P4) > 可延迟(P5) | ✅ 文档描述完整 |
| TC-605 | Playbook 互斥规则 | Expansion 与挽留同时触发 | PB-EXP-001 与 PB-RNW-002 互斥，不能同时追增购又做挽留 | ✅ 文档描述完整 |
| TC-606 | Playbook 过期清理 | in_progress 超过预期时长 1.5 倍 | 自动标记为 stale，通知 CSM；7 天未响应则推送确认：继续/暂停/关闭 | ✅ 文档描述完整 |

### 模块 7：安全与自动化治理

| 用例ID | 测试场景 | 输入/触发条件 | 预期输出 | 测试状态 |
|--------|----------|---------------|----------|----------|
| TC-701 | 数据分级 - S4 绝密数据 | 涉及客户财务数据、人事变动 | 不在 AI 输出中直接展示，仅引用为"敏感信息已脱敏" | ✅ 文档描述完整 |
| TC-702 | 自动化等级边界 - L1 | 客户自动化等级 = L1 | 所有操作需 CSM 确认后执行；AI 仅提供分析和建议 | ✅ 文档描述完整 |
| TC-703 | 自动化等级边界 - L4-L5 | 客户自动化等级 = L5 | 低风险操作可自动执行，高风险操作仍需确认；系统永远不会超过客户被分配的自动化等级 | ✅ 文档描述完整 |
| TC-704 | 优雅降级 - G1 精度降低 | 部分数据缺失 | 用已有数据给出最佳估算，标注降级等级 G1、缺失的数据、补全路径 | ✅ 文档描述完整 |
| TC-705 | 优雅降级 - G3 仅骨架 | 几乎无数据 | 输出标准模板骨架 + CSM 手动填充指引 | ✅ 文档描述完整 |

### 模块 8：跨 Skill 协同与数据流

| 用例ID | 测试场景 | 输入/触发条件 | 预期输出 | 测试状态 |
|--------|----------|---------------|----------|----------|
| TC-801 | Health-Dashboard → Churn-Predict | 健康分跌入黄色/红色 | 自动触发流失预测分析，输出风险因子排序 | ✅ 文档描述完整 |
| TC-802 | Sales-Handoff → TTV-Tracker | 交接完成 | 交接信息中的客户期望、购买原因和成功标准自动成为 TTV 路径的目标定义 | ✅ 文档描述完整 |
| TC-803 | Value-Proof-Generator → Renewal-War-Room | 续约准备阶段 | 价值证明文档（ROI、业务成果）自动注入续约谈判的"价值论据"板块 | ✅ 文档描述完整 |
| TC-804 | S3/S4 数据跨 Skill 传递限制 | 涉及敏感数据 | S3/S4 级数据不跨 Skill 自动传递，需遵循数据分级规则 | ✅ 文档描述完整 |
| TC-805 | 上下文传递关闭 | 用户说"不用之前的分析结果" | 关闭跨 Skill 自动传递功能 | ✅ 文档描述完整 |

### 模块 9：通用输出规范

| 用例ID | 测试场景 | 输入/触发条件 | 预期输出 | 测试状态 |
|--------|----------|---------------|----------|----------|
| TC-901 | 置信度标注 - 高置信 | 基于完整数据和明确规则 | 输出标注"高置信" | ✅ 文档描述完整 |
| TC-902 | 置信度标注 - 低置信 | 基于有限信息的估算 | 输出标注"低置信"，并附加 `[数据不足，建议补充]` | ✅ 文档描述完整 |
| TC-903 | 数据来源标注 | 每个关键结论 | 来自 Obsidian 标注 `[来自知识库]`，来自用户标注 `[来自用户]`，AI 推断标注 `[AI推断]` | ✅ 文档描述完整 |
| TC-904 | 去 AI 味规则 | 输出中包含禁用词 | 禁用"赋能""闭环""抓手""打通""沉淀"等词，用具体描述替代 | ✅ 文档描述完整 |
| TC-905 | 中国业务场景适配 | 节假日/重要时间节点 | 自动感知国庆、春节前后沟通节奏调整；IM 优先（企微/飞书/钉钉） | ✅ 文档描述完整 |

---

## 四、发现的问题与建议

| # | 问题/建议 | 严重程度 | 位置 | 说明 |
|---|----------|---------|------|------|
| 1 | `skill-matrix.md` 中自动化级别定义与 SKILL.md 不一致 | 中 | references/skill-matrix.md vs SKILL.md | SKILL.md 定义 L1-L5 五级自动化；skill-matrix.md 末尾只定义了 L1-L4 四级（缺少 L5），且描述有差异 |
| 2 | Health-Dashboard 适用阶段描述不一致 | 低 | SKILL.md vs skill-matrix.md | SKILL.md 中 Health-Dashboard 适用阶段标注为"全阶段"；skill-matrix.md 中标注为"Onboarding -> Renewal（权重随阶段动态调整）"，未明确包含 Handoff 和 Advocacy |
| 3 | 客户旅程阶段数量不一致 | 低 | SKILL.md vs references/customer-journey.md | SKILL.md 描述为"6 个旅程阶段"；customer-journey.md 实际包含 7 个阶段（增加了 Advocacy 客户代言阶段），文档间未统一 |
| 4 | 健康评分模型权重与 SKILL.md 不一致 | 中 | SKILL.md vs references/health-score-model.md | SKILL.md 中"产品采纳"默认权重 25%、"支持健康"15%、"客户情绪"10%；health-score-model.md 中"产品使用深度"25%、"活跃度趋势"20%、"NPS/满意度"5%，维度名称和权重均有差异 |
| 5 | 17 个子 Skill 的输入/输出规格完备性 | 低 | references/skill-matrix.md | 虽然每个 Skill 都定义了 Inputs/Outputs，但部分字段的数据类型定义较宽松（如 `time_range: string` 未约束格式），实际运行时可能出现解析问题 |
| 6 | Obsidian Vault 依赖的外部工具 | 低 | 整体设计 | 系统重度依赖 Obsidian 作为知识库载体，要求用户安装并配置 Obsidian，对于不使用 Obsidian 的用户可用性受限 |
| 7 | `mcp-integration.md` 中 MCP 外部数据源对接 | 低 | references/mcp-integration.md | 文档提及通过 MCP 对接 CRM API、工单系统等外部数据源，但实际 MCP 配置方式、认证流程、错误处理未在本次审查中验证 |
| 8 | 阶段激活矩阵中部分 Skill 的 Advocacy 阶段标记不一致 | 低 | references/skill-matrix.md vs customer-journey.md | skill-matrix.md 中 Executive-Briefing 在 Advocacy 阶段未标记激活；customer-journey.md 中明确列出 Executive-Briefing 为 Advocacy 阶段激活 Skill |

---

## 五、测试结论

| 维度 | 评分（1-5） | 说明 |
|------|-------------|------|
| 功能完整性 | 5 | 17 个子 Skill 覆盖客户全生命周期，6 阶段客户旅程、3 层分级、7 维健康评分、10 个 Playbook、5 级自动化、3 级降级，功能设计极为全面 |
| 文档质量 | 4 | 整体文档体系完善，知识库丰富；存在少量跨文档不一致（健康评分权重、自动化级别定义、阶段数量等），建议统一 |
| 测试覆盖度 | 4 | 基于文档设计了 40 个测试用例，覆盖引导初始化、路由、客户旅程、分层、健康评分、Playbook、安全治理、跨 Skill 协同、输出规范；部分外部集成和脚本未验证 |
| 可测试性 | 4 | 路由规则明确，输入/输出规格清晰，Playbook 有明确的触发条件和成功指标；但 Obsidian 依赖、MCP 集成、部分数据类型约束松散增加了实际测试复杂度 |

**总体评价**：深信 · 客户成功（Sloth-SkyHub-Eido）v1.0.0 是一款架构先进、设计精细的客户成功智能体矩阵。其客户旅程驱动、Skill-as-Code、优雅降级等设计理念具有显著差异化优势。建议优先修复跨文档一致性 issue，并验证 Obsidian 知识库与 MCP 集成的实际可用性。

---

> 本报告为基于文档的静态审查，未涉及实际 QoderWork 运行时测试。实际功能验证需在安装技能后通过对话交互完成。
