# Supabase Schema Notes

本文件是 Supabase `chatgpt_memory` 数据结构说明，来源于当前项目已使用的 Supabase 表、既有查询模板，以及 PR / issue 讨论中提到的数据库脚本口径整理。

本文件只描述结构和写入边界，不保存具体游戏画像、用户偏好、游玩记录或场景状态数据。

## 数据库定位

```text
project: chatgpt_memory
schema: public
namespace: game_filter
```

Supabase `chatgpt_memory` 不是单纯的游戏画像主库，而是游戏筛选项目的数据运行库。

## 当前数据层分工

```text
public.memory_items = 共享游戏资料层
public.memory_events = 事件 / 审计记录层
public.memory_import_batches = 导入批次层
public.memory_import_file_chunks = 导入文件分块层
public.memory_users = 用户身份层
public.profile_aliases = profile code 路由层
public.user_preference_items = 用户偏好与个人游玩记录层
public.user_scenario_items = 用户场景状态层
```

## 表结构说明

### public.memory_items

用途：保存跨用户共享的游戏画像、共享游戏事实、legacy 原始段落、共享正负面索引和可公共化的结构性事实。

常用字段：

| 字段 | 含义 |
|---|---|
| `id` | 记录主键 |
| `namespace` | 项目命名空间，当前使用 `game_filter` |
| `item_type` | 资料类型，例如 `structured_profile_record`、`raw_legacy_profile_section`、`positive_reference_index`、`negative_decision_index` |
| `item_key` | 稳定条目键，例如游戏 key 或索引 key |
| `title` | 可读标题，常用于游戏名搜索 |
| `payload` | JSON / JSONB 结构化内容 |
| `source` | 来源或导入来源说明 |
| `created_at` | 创建时间 |
| `updated_at` | 更新时间 |

禁止写入：某个用户的已玩记录、个人偏好、个人推荐状态、个人等待 / 排除 / 低优先结论。

### public.memory_events

用途：记录导入、保存、同步、修正等事件，作为低风险审计日志。

常用字段：

| 字段 | 含义 |
|---|---|
| `id` | 事件主键 |
| `namespace` | 项目命名空间 |
| `event_type` | 事件类型，例如 import、save、sync、correction |
| `title` | 事件标题 |
| `payload` | 事件详情 JSON / JSONB |
| `created_at` | 事件时间 |

### public.memory_import_batches

用途：记录一次导入任务或迁移批次。

常用字段：

| 字段 | 含义 |
|---|---|
| `id` | 批次主键 |
| `namespace` | 项目命名空间 |
| `batch_key` | 导入批次键 |
| `source_name` | 来源文件或来源说明 |
| `payload` | 导入统计、参数、校验信息 |
| `created_at` | 创建时间 |

### public.memory_import_file_chunks

用途：保存导入源文件的分块或摘要，辅助追溯 legacy 导入来源。

常用字段：

| 字段 | 含义 |
|---|---|
| `id` | 分块主键 |
| `namespace` | 项目命名空间 |
| `batch_key` | 对应导入批次键 |
| `chunk_index` | 分块序号 |
| `content` | 分块文本或摘要 |
| `payload` | 附加元数据 JSON / JSONB |
| `created_at` | 创建时间 |

### public.memory_users

用途：保存内部用户身份，不保存平台账号、真实身份、密钥或隐私资料。

常用字段：

| 字段 | 含义 |
|---|---|
| `user_key` | 内部用户键，例如 `owner_zhengkun`、`u_123` |
| `display_name` | 低风险显示名 |
| `payload` | 用户元信息 JSON / JSONB |
| `created_at` | 创建时间 |
| `updated_at` | 更新时间 |

### public.profile_aliases

用途：把用户输入的 profile code 路由到内部 `user_key`。

常用字段：

| 字段 | 含义 |
|---|---|
| `alias_norm` | 规范化 profile code，规则为 `lower(trim(profile_code))` |
| `user_key` | 内部用户键 |
| `created_at` | 创建时间 |
| `updated_at` | 更新时间 |

路由真源：`profile_aliases.alias_norm -> user_key`。

### public.user_preference_items

用途：保存某个用户跨场景复用的稳定偏好、反馈覆盖、个人游玩记录、个人正负面参考等。

常用字段：

| 字段 | 含义 |
|---|---|
| `id` | 记录主键 |
| `user_key` | 内部用户键 |
| `namespace` | 项目命名空间，当前使用 `game_filter` |
| `item_type` | 用户偏好条目类型 |
| `item_key` | 稳定条目键，例如 `played:<game_key>`、`pref:<topic>`、`feedback:<game_key>` |
| `title` | 可读标题 |
| `payload` | JSON / JSONB 结构化内容 |
| `created_at` | 创建时间 |
| `updated_at` | 更新时间 |

当前已核实常用 `item_type`：

```text
user_profile_meta
stable_preference
game_feedback_overlay
positive_reference_index
negative_reference_index
played_record
played_record_library_meta
played_record_schema
legacy_personal_raw_section_archive
```

禁止写入：共享游戏画像、公共事实、规则正文、读取顺序、保存流程。

### public.user_scenario_items

用途：保存某个用户在某个 scenario 下对具体游戏或场景定义记录的状态真源。

常用字段：

| 字段 | 含义 |
|---|---|
| `id` | 记录主键 |
| `user_key` | 内部用户键 |
| `namespace` | 项目命名空间，当前使用 `game_filter` |
| `scenario_code` | 场景代号 |
| `game_key` | 游戏 key；场景定义记录可使用 `__scenario_definition__` |
| `title` | 可读标题 |
| `state` | 场景状态字段；必须使用 `state`，不要写成 `status` |
| `reason` | 状态理由摘要 |
| `payload` | JSON / JSONB 结构化内容，包括审计结果、等待条件、来源摘要等 |
| `last_checked_at` | 最近外部事实核查时间 |
| `created_at` | 创建时间 |
| `updated_at` | 更新时间 |

唯一状态键逻辑：

```text
user_key + namespace + scenario_code + game_key
```

场景定义记录：

```text
game_key = __scenario_definition__
state = scenario_definition
```

当前已见 `state` 值：

```text
recommended
waiting
investigate
low_priority
excluded
reference_only
scenario_definition
active_baseline
waiting_recheck
```

禁止写入：跨用户共享事实、规则正文、读取顺序、保存流程、其他用户的偏好或状态。

## 关键命名纠偏

### user_scenario_items 字段名

数据库字段是：

```text
state
```

不要写成：

```text
status
```

中文可以说“状态”，但涉及数据库字段时必须使用 `state`。

### 正负面索引 item_type

用户偏好层 `public.user_preference_items` 当前使用：

```text
positive_reference_index
negative_reference_index
```

共享资料层 `public.memory_items` 当前使用：

```text
positive_reference_index
negative_decision_index
```

不要把 `positive_reference / negative_reference` 写成当前规范 `item_type`。如只是在 payload 内部出现历史词汇，可保留。

## 写入边界

```text
public.memory_items：只写共享资料、公共事实和共享索引。
public.user_preference_items：只写用户跨场景偏好、反馈覆盖和游玩记录。
public.user_scenario_items：只写用户在某场景下的状态、候选、等待、排除、定义记录。
public.memory_events：只写事件 / 审计摘要。
rules/：只保存规则真源，不保存数据。
memory/：只保存可读快照、场景模板和项目状态快照，不保存规则真源。
```

## 推荐前最低读取要求

推荐、筛选、更新清单、解释场景状态之前，必须读取并合并：

```text
1. public.memory_items
2. public.user_preference_items，包括 played_record
3. public.user_scenario_items，字段为 state
4. GitHub memory/scenario_types/*.md 类型模板
5. GitHub memory/profiles/<user_key>/scenarios/*.md 用户场景快照
```

如果缺少 `played_record_lookup_result` 或 `played_record_status_effect`，候选不得进入推荐位。
