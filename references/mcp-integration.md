# MCP集成架构

> Model Context Protocol (MCP) 是AI Agent与外部系统之间的标准化接口协议。SkyHub采用本地MCP Server作为桥接层，实现Skill与数据源的解耦，同时确保每个外部依赖都有本地降级方案。

---

## 架构总览

```
┌──────────────────────────────────────────────────────────┐
│                    QoderWork Agent                        │
│                  (Skill执行引擎)                          │
└────────────────────┬─────────────────────────────────────┘
                     │ MCP Protocol (JSON-RPC over stdio)
        ┌────────────┼────────────────┐
        ▼            ▼                ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│  本地核心MCP  │ │  本地核心MCP  │ │  外部可选MCP  │
│  (常驻)      │ │  (按需)      │ │  (可降级)     │
│ filesystem   │ │ script-runner│ │ crm/wechat/  │
│ sqlite       │ │ csv-reader   │ │ dingtalk/bi  │
└──────────────┘ └──────────────┘ └──────────────┘
        │            │                │
        ▼            ▼                ▼
┌──────────────────────────────────────────────────────────┐
│              Obsidian Vault (SkyHub-KB/)                  │
│              本地SQLite / CSV文件                          │
│              外部SaaS系统                                  │
└──────────────────────────────────────────────────────────┘
```

---

## 本地核心MCP Server

本地核心MCP Server是SkyHub运行的必要基础设施，不可降级。

| MCP Server | 功能 | 模式 | 依赖 | 端口/协议 |
|-----------|------|------|------|----------|
| mcp-filesystem | 读写Obsidian Vault的Markdown/YAML文件 | 本地进程，常驻 | Node.js >= 18 | stdio |
| mcp-script-runner | 执行Skill的Python/Shell脚本 | 本地进程，按需启动 | Python 3.11+ | stdio |
| mcp-sqlite | 查询本地SQLite数据库（健康分历史、操作日志） | 本地进程，常驻 | SQLite3 | stdio |
| mcp-csv-reader | 解析本地CSV/Excel数据文件 | 本地进程，按需启动 | Python 3.11+, pandas | stdio |

### mcp-filesystem 详细说明

- **核心能力**: `read_file`, `write_file`, `list_directory`, `search_files`, `move_file`
- **根目录限制**: 仅允许访问 `SkyHub-KB/` 目录及其子目录，防止越权读写
- **并发控制**: 单写多读，写操作自动加文件锁
- **编码**: 强制UTF-8，确保中文内容正确处理

### mcp-script-runner 详细说明

- **核心能力**: `run_python`, `run_shell`
- **沙箱**: 脚本在受限环境中执行，禁止网络访问（除非显式配置白名单）
- **超时**: 默认30秒，长任务（如批量健康分计算）可配置至300秒
- **输出捕获**: stdout/stderr均捕获并返回Agent

### mcp-sqlite 详细说明

- **核心能力**: `query`, `execute`, `list_tables`, `describe_table`
- **数据库文件**: `SkyHub-KB/_data/skyhub.db`
- **表结构**: `health_score_history`, `operation_log`, `playbook_execution_log`, `voc_metrics`
- **只读模式**: 默认只读，写入操作需显式授权

### mcp-csv-reader 详细说明

- **核心能力**: `read_csv`, `read_excel`, `query_dataframe`
- **支持格式**: CSV, TSV, XLSX, XLS
- **内存限制**: 单文件最大100MB，超出自动分块处理
- **查询能力**: 支持pandas-style查询表达式，如 `df[df['arr'] > 500000]`

---

## 外部可选MCP Server

外部MCP Server提供与第三方系统的集成能力。每个外部Server必须有本地降级方案。

| MCP Server | 功能 | 连接方式 | 降级方案 |
|-----------|------|---------|---------|
| mcp-crm | Salesforce/纷享销客数据同步 | HTTPS API | 本地缓存CRM导出的CSV/JSON，CSM定期手动导出更新 |
| mcp-wechat-work | 企业微信消息发送与接收 | WebSocket | CSM手动复制Agent输出内容粘贴至企微 |
| mcp-dingtalk | 钉钉消息推送与审批流 | HTTPS API | CSM手动操作钉钉，审批结果手动录入 |
| mcp-tianyancha | 天眼查企业信息查询 | HTTPS API | 本地竞争情报库 `04-Knowledge/Competitor_Intel/` |
| mcp-bi | BI系统数据查询（Tableau/PowerBI/Metabase） | HTTPS API | 手动导出CSV放入 `SkyHub-KB/_data/` 目录 |
| mcp-calendar | 日历系统集成（Google/Outlook/飞书） | HTTPS API | 手动在续约日历仪表盘中维护日期 |
| mcp-ticket | 工单系统集成（Zendesk/Jira Service） | HTTPS API | 手动更新客户档案中的support_health维度 |

---

## 降级原则

> **核心理念: 每个外部MCP Server必须有本地降级方案。有损降级优于完全失败。**

### 降级层级

| 层级 | 描述 | 示例 |
|------|------|------|
| L0 - 全功能 | 外部MCP Server正常运行，实时数据同步 | mcp-crm实时拉取Salesforce客户数据 |
| L1 - 缓存降级 | 外部不可用，使用本地缓存数据（标注数据时效） | 使用上次同步的CRM导出数据，标注"数据截止: 2026-04-15" |
| L2 - 手动降级 | 缓存过期，CSM手动操作并录入结果 | CSM手动查询CRM后更新客户Frontmatter |
| L3 - 功能跳过 | 该能力暂时不可用，Skill跳过相关步骤并提示 | "CRM数据不可用，跳过自动客户信息更新步骤" |

### 降级检测与切换

- Skill启动时自动探测所有依赖的MCP Server可用性
- 不可用时自动切换到降级方案，并在输出中标注降级状态
- 降级事件记录到 `_governance/degradation_log.md`

---

## MCP配置模板

QoderWork的MCP Server配置文件示例：

```yaml
# .qoder/mcp-servers.yaml
mcp_servers:
  # === 本地核心（必需） ===
  mcp-filesystem:
    command: "npx"
    args: ["-y", "@anthropic/mcp-filesystem", "/path/to/SkyHub-KB"]
    mode: "persistent"
    restart_policy: "always"

  mcp-script-runner:
    command: "python"
    args: ["-m", "mcp_script_runner", "--sandbox"]
    mode: "on_demand"
    timeout_seconds: 300
    env:
      PYTHONPATH: "/path/to/SkyHub-KB/03-Skills"

  mcp-sqlite:
    command: "npx"
    args: ["-y", "@anthropic/mcp-sqlite", "/path/to/SkyHub-KB/_data/skyhub.db"]
    mode: "persistent"
    read_only: true

  mcp-csv-reader:
    command: "python"
    args: ["-m", "mcp_csv_reader"]
    mode: "on_demand"
    env:
      MAX_FILE_SIZE_MB: "100"

  # === 外部可选（按需启用） ===
  mcp-crm:
    command: "node"
    args: ["mcp-crm-bridge/index.js"]
    mode: "on_demand"
    env:
      CRM_TYPE: "salesforce"           # salesforce | fenxiang
      CRM_API_BASE: "${CRM_API_BASE}"
      CRM_API_TOKEN: "${CRM_API_TOKEN}"
    fallback:
      type: "local_cache"
      cache_dir: "SkyHub-KB/_data/crm_cache/"
      max_age_hours: 72

  mcp-wechat-work:
    command: "node"
    args: ["mcp-wechat-work/index.js"]
    mode: "on_demand"
    env:
      WECHAT_CORP_ID: "${WECHAT_CORP_ID}"
      WECHAT_AGENT_ID: "${WECHAT_AGENT_ID}"
      WECHAT_SECRET: "${WECHAT_SECRET}"
    fallback:
      type: "manual"
      instruction: "请手动将以下内容发送至企业微信"

  mcp-dingtalk:
    command: "node"
    args: ["mcp-dingtalk-bridge/index.js"]
    mode: "on_demand"
    env:
      DINGTALK_APP_KEY: "${DINGTALK_APP_KEY}"
      DINGTALK_APP_SECRET: "${DINGTALK_APP_SECRET}"
    fallback:
      type: "manual"
      instruction: "请手动在钉钉中完成操作"
```

---

## 数据流向

```
外部系统 (CRM/BI/工单)
        │
        ▼ (MCP外部Server拉取)
  MCP Server层 (数据标准化)
        │
        ▼ (JSON-RPC调用)
  Skill执行层 (业务逻辑处理)
        │
        ├──▶ mcp-filesystem ──▶ Obsidian Vault (Markdown/YAML)
        ├──▶ mcp-sqlite     ──▶ 本地SQLite (时序数据)
        └──▶ mcp-csv-reader  ◀── 本地CSV/Excel (批量导入)
```

### 数据同步策略

| 数据类型 | 同步频率 | 方向 | 冲突策略 |
|---------|---------|------|---------|
| 客户基本信息 | 每日/手动触发 | CRM -> Vault | CRM为权威源，覆盖本地 |
| 健康分数据 | 实时计算 | Vault内部 | 最新计算结果覆盖 |
| VoC数据 | 事件驱动/批量导入 | 外部 -> Vault | 追加写入，不覆盖 |
| 报告文档 | Skill生成时写入 | Skill -> Vault | 版本命名，不覆盖 |

---

## 错误处理

### 重试策略

```yaml
retry_policy:
  max_retries: 2
  interval_seconds: 5
  backoff_multiplier: 2        # 第一次5s, 第二次10s
  retryable_errors:
    - "TIMEOUT"
    - "CONNECTION_REFUSED"
    - "RATE_LIMITED"
  non_retryable_errors:
    - "AUTH_FAILED"
    - "PERMISSION_DENIED"
    - "NOT_FOUND"
```

### 熔断器 (Circuit Breaker)

```yaml
circuit_breaker:
  failure_threshold: 3          # 连续3次失败触发熔断
  recovery_timeout_seconds: 300 # 熔断后5分钟尝试恢复
  half_open_max_calls: 1        # 半开状态最多1次试探
```

- **关闭状态**: 正常转发请求到MCP Server
- **打开状态**: 直接返回降级结果，不再尝试调用MCP Server
- **半开状态**: 允许一次试探调用，成功则关闭熔断，失败则重新打开

### 降级级联

```
MCP Server调用失败
    ├── 重试 (2次, 指数退避)
    │   ├── 成功 → 正常返回
    │   └── 失败 → 熔断器检查
    │       ├── 未熔断 → 记录失败计数
    │       └── 已熔断 → 降级处理
    └── 降级处理
        ├── L1: 读取本地缓存
        ├── L2: 提示CSM手动操作
        └── L3: 跳过步骤并标注
```

---

## 安全

### MCP Server权限控制

| 权限维度 | 控制方式 | 说明 |
|---------|---------|------|
| 文件系统访问 | 根目录白名单 | mcp-filesystem仅可访问 `SkyHub-KB/` |
| 脚本执行 | 沙箱 + 白名单 | mcp-script-runner禁止任意命令，仅允许注册的Skill脚本 |
| 数据库操作 | 只读模式默认 | mcp-sqlite默认只读，写入需Skill显式声明 |
| 网络访问 | 出站白名单 | 外部MCP Server仅可访问其声明的API域名 |
| 环境变量 | 加密存储 | API Token等敏感信息通过环境变量注入，不写入配置文件 |

### 数据分类在MCP层的强制执行

| 数据等级 | 定义 | MCP层处理 |
|---------|------|----------|
| P0 - 公开 | 产品文档、FAQ | 无限制读写 |
| P1 - 内部 | 客户名称、行业、Tier | 读取无限制，写入需Skill声明 |
| P2 - 敏感 | ARR、合同金额、联系方式 | 读取需权限标记，输出自动脱敏 |
| P3 - 机密 | API密钥、内部战略文档 | MCP层拦截，不传递给Agent |

### 脱敏规则

- **P2字段输出脱敏**: ARR显示为区间（如"100-500万"），联系电话后4位用 `****` 替代
- **脱敏配置文件**: `_config/data_masking_rules.yaml` 定义具体规则
- **脱敏执行点**: mcp-filesystem在读取文件时，根据调用方的权限等级自动应用脱敏规则

### 审计日志

所有MCP Server调用记录写入 `_governance/skill_audit_log.md`，包含：
- 调用时间、调用方Skill、目标MCP Server
- 操作类型（read/write/execute）
- 涉及的数据等级
- 成功/失败状态
- 降级事件标记
