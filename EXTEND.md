# Sloth-SkyHub-Eido 用户个性化配置

> 所有配置项均为可选，留空或保持默认值即可正常使用。

---

## 中央知识库集成（可选）

```yaml
# ===== 中央知识库集成（Sloth-KnowledgeHub-Eido）=====
# 启用后，SkyHub 在知识结晶点自动萃取知识推送到中央库，
# 在知识需求点自动查询中央库并展示推荐。
# 详见 modules/knowledge-hub-bridge.md
knowledge_hub:
  enabled: false                          # 是否启用 Hub 集成
  vault_path: "/Users/wangguobao/Obsidian/Sloth-Central-Vault"  # ⚠️ 请修改为你本机的实际 Vault 路径
  auto_publish_confidence: B              # 自动萃取的最低置信度
  query_result_limit: 3                   # 推荐结果上限
```
