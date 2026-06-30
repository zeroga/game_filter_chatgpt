# 游戏筛选项目工作档

更新日期：2026-07-01（JST）

## 1. 当前定位

本仓库用于保存 ChatGPT 可读取的项目工作档记忆，不保存完整游戏画像数据库。

当前分工：

| 层 | 职责 |
|---|---|
| GitHub `zeroga/game_filter_chatgpt` | 工作档、当前状态、读写规则、数据库说明、变更记录 |
| Supabase `chatgpt_memory` | 游戏画像数据主库，供 ChatGPT / GPT Action 直接查询和写入 |
| 上传 TXT / SQL 文件 | 一次性迁移源或备份，不作为日常主读取源 |

## 2. 当前规则基线

项目规则基线为：

```text
v4.6_game_filter_preference_library_machine.txt
```

其角色为：

```text
rules_schema_only
```

关键规则：

- v4.6 只定义规则、schema、状态机、审计要求、输出约束。
- 完整游戏画像不存放在规则文件中。
- 游戏画像数据必须从数据库读取。
- 自然语言说明只是注释，机器字段优先。
- silent load 只控制外部输出，不减少内部读取、索引、审计和自检。
- 推荐目标是上限，不是填满义务；干净候选不足时保留空位。
- 推荐前必须执行候选审计。
- 若缺游戏数据库或候选审计，阻断推荐。

## 3. 当前数据主库

数据主库：Supabase `chatgpt_memory`

```text
project_ref: zpkfrbfgaaojrblcqvvl
schema: public
namespace: game_filter
main_table: memory_items
event_table: memory_events
import_batch_table: memory_import_batches
```

已迁入：

```text
v1.0_game_filter_game_profile_database_machine.txt
```

导入结果：

```text
memory_items: 404 rows
structured_profile_record: 368 rows
raw_legacy_profile_section: 28 rows
memory_events: 1 row
memory_import_batches: 1 row
```

## 4. 直接连库读写原则

日常游戏筛选时：

1. 先读取本工作档入口和当前状态。
2. 按规则确认需要的数据范围。
3. 查询 Supabase 共享游戏资料层：`public.memory_items`。
4. 查询 Supabase 用户偏好层：`public.user_preference_items`。
5. 查询 Supabase 用户场景层：`public.user_scenario_items`。
6. 涉及当前事实、价格、版本、评价、发售状态、DLC、联机结构时联网核查。
7. 不把数据库全量内容回写 GitHub。

## 5. 写入原则

写入 GitHub：

- 项目状态变化
- 当前任务续接点
- 规则说明变化
- 数据库结构说明变化
- 变更日志

写入 Supabase：

- 共享游戏事实和通用游戏画像
- 用户稳定偏好
- 用户游玩记录
- 用户反馈覆盖
- 用户场景状态
- 场景条目
- 待查任务
- 等待条件
- 来源摘要

禁止写入：

- OAuth token
- API key
- service role key
- 私密账号信息
- 真实身份信息
- 不应公开的用户隐私

## 6. 个人游玩记录库机制

个人游玩记录库位于用户偏好层：

```text
public.user_preference_items
user_key = owner_zhengkun
namespace = game_filter
```

当前已建立的库元数据：

```text
item_type = played_record_library_meta
item_key = current
```

当前已建立的字段规范：

```text
item_type = played_record_schema
item_key = v1
```

每个游戏的个人游玩记录使用：

```text
item_type = played_record
item_key = played:<game_key>
```

### 6.1 `played_record` 必填字段

```text
game_key
game_name
play_status
source_confidence
last_updated_jst
```

### 6.2 `played_record` 建议字段

```text
platforms_played
hours_band
completion_state
coop_experience
positive_points
negative_points
drop_reason
revisit_condition
related_scenarios
evidence_source
notes
```

### 6.3 `play_status` 枚举

```text
played
currently_playing
completed
fully_completed
abandoned
refunded
tried_negative
strong_positive_reference
strong_negative_reference
reference_only
```

### 6.4 `source_confidence` 枚举

```text
user_firsthand_explicit
user_firsthand_memory
imported_legacy_workdoc
assistant_inferred_needs_confirmation
```

SteamDB 占位记录或未确认的历史推断记录必须标记：

```text
source_confidence = assistant_inferred_needs_confirmation
needs_user_confirmation = true
```

### 6.5 个人索引和偏好

用户稳定偏好写入：

```text
public.user_preference_items
item_type = stable_preference
```

用户负面参考索引写入：

```text
public.user_preference_items
item_type = negative_reference_index
```

用户正面参考索引写入：

```text
public.user_preference_items
item_type = positive_reference_index
```

legacy 原文归档写入：

```text
public.user_preference_items
item_type = legacy_personal_raw_section_archive
item_key = raw_section:<legacy_section_key>
```

## 7. 用户场景状态机制

用户在某个场景下对某游戏的推荐、待查、等待、排除、低优先、参考、基准线状态写入：

```text
public.user_scenario_items
user_key = owner_zhengkun
namespace = game_filter
scenario_code = <scenario_code>
game_key = <game_key>
```

当前存在一个历史导入暂存场景：

```text
scenario_code = legacy_imported_status
```

`legacy_imported_status` 只表示历史状态暂存区，不是正式推荐场景。后续需要按真实 `scenario_code` 重新分类。

## 8. 已验证样本

数据库直连已验证可用。样本查询结果包括：

- `Warframe`：正面参考，长线 PvE / Mod / 武器和战甲收集。
- `No Man's Sky`：单人正面、联机同步负面。
- `Stolen Realm`：用户已拒绝，负面参考。
- `Inkbound`：当前正面观察。
- `Granblue Fantasy: Relink`：已通关等待大型内容。
- `X4: Foundations`：单人太空 / 舰队经营强正面；因无官方联机，不进入 PC/主机联机场景推荐。

这些样本如果涉及个人体验、已玩状态、正负面参考或场景结论，应从用户层合并读取，而不是只看共享游戏资料层。

## 9. 安全边界

`chatgpt_memory` 为降低直连授权摩擦，当前 RLS 有意关闭。

这意味着：

- 数据库只应存放可公开或低风险的游戏筛选数据。
- 不应存放密钥、token、私密账户数据或不可公开的个人信息。
- GitHub 中不保存 Supabase publishable key 或 anon key。
- 用户层可以保存低风险游戏偏好和游玩记录，但不能保存账号、交易、好友、真实身份、密钥或平台隐私。

## 10. 下次续接方式

新对话继续游戏筛选时，优先读取：

```text
memory/index.json
memory/current_state.md
memory/schema_notes.md
memory/project_workdoc.md
```

然后查询 Supabase 共享游戏资料层：

```sql
select *
from public.memory_items
where namespace = 'game_filter'
  and item_type = 'structured_profile_record';
```

再查询用户偏好层：

```sql
select *
from public.user_preference_items
where user_key = 'owner_zhengkun'
  and namespace = 'game_filter';
```

再查询用户场景层：

```sql
select *
from public.user_scenario_items
where user_key = 'owner_zhengkun'
  and namespace = 'game_filter';
```

如果是特定游戏，三个层都要按 `title / game_key / item_key / payload` 查询，不能只查 `public.memory_items`。
