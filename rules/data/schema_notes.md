# Supabase Schema Notes

本文件是 Supabase `chatgpt_memory` 数据结构说明，按 Issue #3 评论中的数据库脚本口径整理，并结合当前项目已使用的 Supabase 表和既有查询模板补充写入边界。

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
public.user_report_subscriptions = profile 级周报配置、调度映射和运行快照层
```

## Issue #3 评论数据库脚本映射

Issue #3 评论中的数据库脚本对应的数据结构，应在文档中落到以下运行表和边界：

| 脚本表 / 运行表 | 数据层职责 | 是否保存规则真源 | 说明 |
|---|---|---|---|
| `public.memory_items` | 共享游戏资料层 | 否 | 保存共享游戏画像、legacy 游戏资料段落、共享索引、通用游戏事实和共享游戏资料导入记录 |
| `public.memory_events` | 事件 / 审计层 | 否 | 保存导入、保存、同步、修正等事件摘要 |
| `public.memory_import_batches` | 导入批次层 | 否 | 保存 legacy 游戏资料等导入批次信息 |
| `public.memory_import_file_chunks` | 导入文件分块层 | 否 | 保存导入源文件分块或摘要，辅助追溯游戏资料来源 |
| `public.memory_users` | 用户身份层 | 否 | 保存内部 `user_key` 与低风险显示信息 |
| `public.profile_aliases` | profile code 路由层 | 否 | 保存 `alias_norm -> user_key` 映射 |
| `public.user_preference_items` | 用户偏好 / 游玩记录层 | 否 | 保存用户稳定偏好、反馈覆盖、played_record、个人正负面参考 |
| `public.user_scenario_items` | 用户场景状态层 | 否 | 保存 `user_key + namespace + scenario_code + game_key` 下的场景状态真源，字段使用 `state` |

评论脚本的结构说明不得被解释为允许 Supabase 保存规则真源。Supabase 仍只保存游戏筛选运行数据，不保存规则正文、读取顺序、保存流程或规则治理逻辑。

## 表结构说明

### public.memory_items

用途：保存跨用户共享的游戏画像、共享游戏事实、legacy 原始段落、共享正负面索引、可公共化的结构性事实和共享游戏资料导入记录。

真实字段：

| 字段 | 含义 |
|---|---|
| `id` | 记录主键 |
| `namespace` | 项目命名空间，当前使用 `game_filter` |
| `item_type` | 资料类型，例如 `structured_profile_record`、`raw_legacy_profile_section`、`positive_reference_index`、`negative_decision_index` |
| `item_key` | 稳定条目键，例如游戏 key 或索引 key |
| `title` | 可读标题，常用于游戏名搜索 |
| `payload` | JSON / JSONB 结构化内容 |
| `raw_text` | 原始文本或 legacy 资料片段 |
| `source_file` | 来源文件名或导入源文件 |
| `source_section` | 来源文件中的章节 / 段落标识 |
| `record_hash` | 记录内容 hash，用于去重或导入校验 |
| `created_at` | 创建时间 |
| `updated_at` | 更新时间 |

禁止写入：某个用户的已玩记录、个人偏好、个人推荐状态、个人等待 / 排除 / 低优先结论。

### public.memory_events

用途：记录导入、保存、同步、修正等事件，作为低风险审计日志。

真实字段：

| 字段 | 含义 |
|---|---|
| `id` | 事件主键 |
| `namespace` | 项目命名空间 |
| `event_type` | 事件类型，例如 import、save、sync、correction |
| `target_type` | 事件作用对象类型，例如 table、item、scenario、preference |
| `target_key` | 事件作用对象键 |
| `before_payload` | 变更前 JSON / JSONB 摘要 |
| `after_payload` | 变更后 JSON / JSONB 摘要 |
| `note` | 事件备注 |
| `created_at` | 事件时间 |

### public.memory_import_batches

用途：记录一次导入任务或迁移批次。

真实字段：

| 字段 | 含义 |
|---|---|
| `id` | 批次主键 |
| `namespace` | 项目命名空间 |
| `source_file` | 来源文件名 |
| `source_version` | 来源版本 |
| `import_status` | 导入状态 |
| `item_count` | 导入条目数 |
| `note` | 导入备注 |
| `created_at` | 创建时间 |

### public.memory_import_file_chunks

用途：保存导入源文件的分块文本，辅助追溯 legacy 游戏资料来源。

真实字段：

| 字段 | 含义 |
|---|---|
| `namespace` | 项目命名空间 |
| `source_file` | 来源文件名 |
| `source_version` | 来源版本 |
| `chunk_index` | 分块序号 |
| `chunk_text` | 分块文本 |
| `created_at` | 创建时间 |

复合主键：

```text
namespace + source_file + source_version + chunk_index
```

### public.memory_users

用途：保存内部用户身份，不保存平台账号、真实身份、密钥或隐私资料。

真实字段：

| 字段 | 含义 |
|---|---|
| `user_key` | 内部用户键，例如 `owner_zhengkun`、`u_123` |
| `display_name` | 低风险显示名 |
| `status` | 用户记录状态 |
| `notes` | 低风险备注 |
| `created_at` | 创建时间 |
| `updated_at` | 更新时间 |

### public.profile_aliases

用途：把用户输入的 profile code 路由到内部 `user_key`。

真实字段：

| 字段 | 含义 |
|---|---|
| `alias_norm` | 规范化 profile code，规则为 `lower(trim(profile_code))` |
| `user_key` | 内部用户键 |
| `label` | alias 可读标签或说明 |
| `created_at` | 创建时间 |
| `updated_at` | 更新时间 |

路由真源：`profile_aliases.alias_norm -> user_key`。

### public.user_preference_items

用途：保存某个用户跨场景复用的稳定偏好、反馈覆盖、个人游玩记录、个人正负面参考等。

真实字段：

| 字段 | 含义 |
|---|---|
| `id` | 记录主键 |
| `user_key` | 内部用户键 |
| `namespace` | 项目命名空间，当前使用 `game_filter` |
| `item_type` | 用户偏好条目类型 |
| `item_key` | 稳定条目键，例如 `played:<game_key>`、`pref:<topic>`、`feedback:<game_key>` |
| `title` | 可读标题 |
| `payload` | JSON / JSONB 结构化内容 |
| `raw_text` | 原始文本或人工备注原文 |
| `source` | 来源说明 |
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

真实字段：

| 字段 | 含义 |
|---|---|
| `id` | 记录主键 |
| `user_key` | 内部用户键 |
| `namespace` | 项目命名空间，当前使用 `game_filter` |
| `scenario_code` | 场景代号 |
| `game_key` | 游戏 key；正式场景定义记录必须使用 `__scenario_definition__` |
| `state` | 场景状态字段；必须使用 `state`，不要写成 `status` |
| `title` | 可读标题 |
| `reason` | 状态理由摘要 |
| `payload` | JSON / JSONB 结构化内容，包括审计结果、等待条件、来源摘要等 |
| `last_checked_at` | 最近外部事实核查时间 |
| `created_at` | 创建时间 |
| `updated_at` | 更新时间 |

唯一状态键逻辑：

```text
user_key + namespace + scenario_code + game_key
```

正式场景定义记录：

```text
game_key = __scenario_definition__
state = scenario_definition
```

每个正式 `user_key + namespace + scenario_code` 必须存在且只能存在一条上述定义记录。新场景创建必须同步写入该记录。周报只据此枚举正式场景；普通游戏记录、场景模板、GitHub 快照、`legacy_imported_status` 都不是场景存在性真源。旧场景缺失时须经用户当次确认后回填并回查，不得自动推断或补写。

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

### public.user_report_subscriptions

用途：保存每个 profile 的周报业务配置、预期调度、实际 Automation 映射、同步/删除恢复状态，以及最近一次经用户确认保存的增量比较快照。它不授予定时任务自动写入权限。

| 字段 | 含义 |
|---|---|
| `id` | 记录主键 |
| `user_key` | 内部用户键 |
| `namespace` | 项目命名空间，当前使用 `game_filter` |
| `report_type` | 第一版固定为 `weekly_game_report` |
| `enabled` | 是否启用 |
| `timezone` | IANA 时区 |
| `schedule_ical` | 预期 iCalendar 调度 |
| `delivery_mode` | 每周必发或无重要内容时跳过 |
| `detail_scope` | JSONB；详细搜索的场景与平台范围 |
| `module_config` | JSONB；新游雷达、EA/试玩/测试、中文、资讯类别、价格促销及传闻开关 |
| `automation_id` | 实际 ChatGPT Automation 映射 |
| `sync_state` | 配置与实际任务的同步状态 |
| `report_period` | 最近一次经当次确认写入的稳定计划周期；按 `schedule_ical` 与订阅 `timezone` 计算，同订阅同周期重试复用 |
| `report_generated` | 对应 `report_period` 是否已生成；如落库必须属于当期展示并获确认范围 |
| `report_written` | 用户明确处理本期建议后，其当次确认写入是否全部成功；不表示接受全部建议，无确认写入时不得伪装为成功 |
| `last_snapshot` | JSONB；最近一次实际获确认并成功写入的周报内容，不含未确认、拒绝或失败项目 |
| `last_run_at` | 最近一次经确认记录的生成时间；不得在只读生成阶段自动更新 |
| `last_success_at` | 最近一次经确认记录的成功写入时间 |
| `last_error` | 最近错误摘要 |
| `created_at` / `updated_at` | 创建和更新时间 |

唯一逻辑键：

```text
user_key + namespace + report_type
```

第一版不使用 `report_code`，不新增 `runtime_write_authorized` 或同类长期授权字段，不新增长期 `user_report_runs` 表。同一逻辑键只能存在一条配置。新表必须从创建时启用 RLS，且不得向 `anon` 或普通 `authenticated` 开放，由受控服务连接执行。

`report_period` 的规范值为 `weekly_game_report:<周期计划触发日在订阅时区的 YYYY-MM-DD>`。计划周期边界按订阅 IANA `timezone` 解释 `schedule_ical`；同一订阅同一周期的首次运行、失败重试、重复触发和重新投递复用同一值，只有下一计划周期生成新值。实际报告身份必须使用稳定订阅记录 `id`（并以 `user_key + namespace + report_type` 校验）与 `report_period` 组合，不得把 `report_period` 单独作为跨订阅唯一键；不同订阅的相同周期不得互相覆盖，也不得为此新增运行历史表。

每期生成阶段对所有系统只读。任何字段，包括 `report_generated`、`report_written`、`last_snapshot`、`last_run_at`、`last_success_at`、`last_error` 和 `sync_state`，如需写入，都必须列入当期完整建议写入内容并获用户当次确认。`report_written = true` 仅表示当次明确确认写入的全部项目均成功，不表示用户接受全部建议；任一确认写入失败或用户全部拒绝且没有确认写入时均不得为 `true`。`last_snapshot` 只能纳入本期实际获确认并成功写入的内容，未确认、明确拒绝或失败项目不得进入。无需第三个处理标志或独立历史表。

`sync_state` 至少应能表达正常同步、不同步、`pending_delete`、删除失败和已停用/删除等状态。删除流程先持久化可恢复的 `pending_delete`，再删除并回查 Automation；成功后才删除或停用订阅。失败时保留订阅、`automation_id` 和 `last_error`，以支持从失败步骤重试或取消删除后恢复，不得产生无法追踪的孤立任务。

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

## RLS 与安全边界

当前 `public` 表 RLS 均为 `false`。

因此该库只能保存低风险游戏筛选数据，不得保存：

```text
密钥
token
真实账号隐私
交易信息
高风险个人信息
```

是否启用 RLS、如何设计权限和白名单，必须另起安全 issue；本 PR 不自动处理 RLS 或权限策略。

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
