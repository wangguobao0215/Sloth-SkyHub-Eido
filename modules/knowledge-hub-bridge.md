# Sloth-SkyHub-Eido Knowledge Hub Bridge

> 客户成功指挥塔 × 中央知识库集成协议（v1.1.0）
> 仅当 EXTEND.md 中 `knowledge_hub.enabled = true` 时生效。

---

## 知识结晶点（Skill → Hub）

当以下操作完成后，AI **应当**向用户展示预填好的萃取建议卡片，经用户确认后写入中央库。

> v1.0 阶段仅激活 `priority: high` 的结晶点。`medium`/`low` 级别为 opt-in，用户可在 EXTEND.md 中配置。

| 触发条件 | 产出 asset_type | 目标 stage | priority | 萃取要素 |
|---------|----------------|-----------|----------|---------|
| Playbook 成功执行（达到预设成功标准） | `playbook` | `customer_success` | **high** | Playbook 名称、执行步骤、成功指标、关键调整 |
| 续约策略制定完成 | `best_practice` | `customer_success` | medium | 续约策略、客户画像、关键动作、成功条件 |
| 客户成功故事沉淀 | `case_study` | `customer_success` | **high** | 客户背景、使用旅程、价值实现、量化成果 |
| 健康度救援成功 | `playbook` | `customer_success` | **high** | 救援触发条件、干预措施、恢复过程、经验总结 |
| 流失分析完成 | `lesson_learned` | `customer_success` | medium | 流失客户画像、流失信号、根因、预防建议 |

### 萃取规则

> 工作流：AI 检测到结晶点 → 预填萃取卡片（含 title/summary/confidence） → 用户确认或微调 → 提交注册。用户可选择「跳过本次」但系统会在下次相同场景再次提示。

1. **Playbook 执行 → playbook**：
   - `title`："{Playbook 名称} 执行记录 - {客户/场景}"
   - `summary`：执行效果和关键调整策略
   - `industry`：从客户行业提取
   - `confidence`：A（达到预设成功指标）; B（部分达标）
   - `content`：Playbook 目标 → 执行步骤 → 关键调整 → 结果对比 → 可优化环节
   - `applicable_skills`：`[Sloth-Sales-Eido, Sloth-DeliveryMatrix-Eido]`

2. **续约策略 → best_practice**（medium）：
   - `title`："{客户类型} 续约最佳实践"
   - `summary`：续约关键动作和成功条件
   - `customer_type`：从客户画像提取
   - `tags`：`[renewal, customer-success]`
   - `applicable_skills`：`[Sloth-Sales-Eido]`

3. **成功故事 → case_study**：
   - `title`："{客户名} 客户成功故事"
   - `summary`：核心价值实现和量化成果
   - `confidence`：A（有量化成果数据）; B（定性描述）
   - `content`：客户背景 → 使用场景 → 价值实现过程 → 量化成果 → 客户证言
   - `applicable_skills`：`[Sloth-Sales-Eido, Sloth-PSC-Eido, Sloth-MGO-Eido]`

4. **健康度救援 → playbook**：
   - `title`："{风险类型} 客户健康救援 Playbook"
   - `summary`：救援触发信号和有效干预方案
   - `tags`：`[health-rescue, churn-prevention]`

5. **流失分析 → lesson_learned**（medium）：
   - `title`："{客户类型} 客户流失分析"
   - `summary`：流失根因和预防建议
   - `confidence`：B（含客户反馈）; C（纯内部推测）
   - `applicable_skills`：`[Sloth-Sales-Eido, Sloth-HybridDev-Eido]`

---

## 知识需求点（Hub → Skill）

当用户启动以下操作时，AI **应当**先查询中央库获取相关知识推荐。

| 触发条件 | 查询维度 | 上下文信号 |
|---------|---------|----------|
| 启动 Playbook 选择/执行 | playbook (customer_success) + best_practice | 客户健康度、客户行业、当前场景 |
| 启动续约准备 | best_practice (renewal) + case_study (success stories) | 客户行业、合同规模、历史健康度 |
| 启动健康度救援 | playbook (health rescue) + lesson_learned (churn) + risk_pattern | 风险类型、客户行业 |
| 启动客户价值评审 | case_study (same industry) + product_brief | 客户行业、使用产品 |

### 推荐展示规则

> 有匹配结果时展示推荐卡片（最多 3 条），用户可选择查看详情或直接继续。无匹配则静默跳过。

1. **Playbook 选择**：优先推荐同类场景下成功执行过的 Playbook，加速决策。
2. **续约准备**：推荐续约最佳实践和成功故事，为续约谈判提供弹药。
3. **健康度救援**：推荐历史救援 Playbook 和流失分析教训，快速找到干预方案。
4. **价值评审**：推荐同行业客户成功案例和产品简报，构建价值叙事。

---

## 反馈通道

当用户在本 Skill 中引用了 Hub 推荐的知识资产后，AI 应当在任务完成时询问："这条知识对你有帮助吗？（有用 / 过时 / 有误）"，并通过 `record_feedback.py` 记录反馈，驱动知识质量自动演化。
